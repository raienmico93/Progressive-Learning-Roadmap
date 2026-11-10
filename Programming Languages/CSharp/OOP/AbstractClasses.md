# C# Abstract Classes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **abstract class** is a class that cannot be instantiated directly and is intended to serve as a **base class** for other classes. It may contain a mix of **abstract members** (declared without implementation, which derived classes must implement) and **concrete members** (fully implemented methods, properties, fields, and constructors) that provide shared functionality to derived classes.

**Technical Definition**
The `abstract` modifier indicates that the target has a missing or incomplete implementation. When applied to a class, it specifies that the class is intended only to be a base class and cannot be instantiated on its own. Non-abstract classes that derive from an abstract class **must implement** all members marked as abstract. Abstract classes can contain both abstract members (which have no implementation and must be overridden) and fully implemented members (such as regular methods, properties, and constructors), allowing them to provide common functionality while still requiring derived classes to implement specific abstract members. Abstract classes cannot be marked `sealed`, as they are designed to be extended. They can contain fields, constructors, and any access modifier on their members, but they support only single inheritance—a class can inherit from exactly one abstract class.

**Beginner-Friendly Explanation**
An abstract class is a **partial blueprint** for a family of related classes. It is like a recipe that has some steps already filled in and some steps left blank for you to complete. You cannot cook the recipe as-is (you cannot instantiate the abstract class), but any concrete recipe that uses it as a base must fill in the blank steps. For example, an abstract `Vehicle` class might provide a `GetInfo()` method that all vehicles share, while leaving `Move()` and `MaxSpeed` blank for `Car` and `Boat` to implement in their own way. Abstract classes sit between **interfaces** (pure contracts) and **concrete classes** (fully implemented types), giving you the ability to share code while still enforcing a contract on derived classes.

---

### Key Characteristics

- **Cannot be instantiated** — Abstract classes cannot be created with `new`; they exist only to be inherited from.
- **Mixed members** — Can contain abstract members (no body, must be overridden) and concrete members (fully implemented, inherited as-is).
- **Single inheritance** — A class can inherit from only one abstract class, unlike interfaces, which support multiple implementation.
- **State and constructors** — Can contain instance fields and constructors, enabling structured state sharing among subclasses.
- **Versioning advantage** — New concrete methods can be added to an abstract class without breaking existing derived classes.
- **`sealed` incompatibility** — Abstract classes cannot be `sealed`, because they are designed for extension.
- **`abstract` modifier** — Applied to classes, methods, properties, indexers, and events to indicate missing or incomplete implementation.

---

### Prerequisites

- Understanding of C# classes, methods, properties, and fields.
- Familiarity with inheritance and the `virtual`/`override` keywords.
- Knowledge of access modifiers (`public`, `protected`, `private`, `internal`).
- Awareness of constructors and constructor chaining (`base(...)`).
- Basic understanding of polymorphism and interfaces.

---

### Related Programming Areas

- **Object-oriented programming** — Abstract classes are a cornerstone of inheritance-based design.
- **Polymorphism** — Abstract methods enable runtime polymorphic dispatch through base-type references.
- **Design patterns** — Template Method, Factory Method, and Strategy patterns often rely on abstract classes.
- **API design** — Abstract base classes provide extension points and shared implementation in frameworks.
- **Composition over inheritance** — Abstract classes should be used judiciously; composition is often preferred for flexibility.

---

### Core Concepts / Features

1. Abstract Classes
2. `abstract` Keyword
3. Abstract Methods
4. Abstract Properties
5. Concrete Methods
6. Abstract Class vs. Interface
7. Abstract Class Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Abstract Classes

### Definitions

**Core Definition**
An abstract class is a class that cannot be instantiated and is designed to be inherited by other classes, providing a mix of abstract and concrete members.

**Technical Definition**
An abstract class is a special type of class that cannot be instantiated directly and serves as a blueprint for other classes. It can contain fields and properties and various kinds of methods: abstract methods (declared without implementation, leaving it to derived classes to provide specific functionality), concrete methods (fully implemented, shared by all subclasses), and virtual methods (default implementation that subclasses can optionally override). Abstract classes support single inheritance and can contain constructors, fields, and members with any access modifier.

**Beginner-Friendly Explanation**
An abstract class is a **base class that is too generic to exist on its own**. It provides common structure and partial implementation for its children.

---

### Purposes

- To provide shared code among related classes while enforcing a common structure.
- To define a contract (through abstract members) that derived classes must fulfill.
- To model concepts that do not make sense as standalone objects (e.g., `Shape`, `Vehicle`, `Entity`).
- To offer default implementations while requiring specific methods to be overridden.
- To serve as a stable extension point in frameworks and libraries.

---

### Syntax Rules and Structure

```
[access modifier] abstract class ClassName
{
    // fields, constructors, concrete members, abstract members
}
```

**Component breakdown**

- `abstract` — The modifier indicating the class cannot be instantiated.
- `ClassName` — The class identifier.
- Body — Can contain any combination of concrete and abstract members.

#### Syntax Rules

- Abstract classes cannot be instantiated with `new`.
- Abstract classes cannot be `sealed`.
- Non-abstract derived classes must implement all inherited abstract members.
- Abstract classes can have constructors (typically `protected`).

#### Constraints and Limitations

- Single inheritance only: a class can inherit from at most one abstract class.
- Abstract classes cannot be used with the `new()` generic constraint if they have abstract members (the constraint requires a public parameterless constructor).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Abstract Class with Mixed Members

```csharp
using System;

public abstract class Vehicle
{
    protected string _brand;

    public Vehicle(string brand) => _brand = brand;

    public string GetInfo() => $"This is a {_brand} vehicle.";
    public virtual void StartEngine() => Console.WriteLine($"{_brand} engine is starting...");
    public abstract void Move();
    public abstract int MaxSpeed { get; }
}

public class Car : Vehicle
{
    public Car(string brand) : base(brand) { }
    public override void Move() => Console.WriteLine($"{_brand} car is driving on the road.");
    public override int MaxSpeed => 200;
}

class Program
{
    static void Main()
    {
        Car car = new Car("Toyota");
        Console.WriteLine(car.GetInfo());
        car.StartEngine();
        car.Move();
        Console.WriteLine($"Max speed: {car.MaxSpeed} km/h");
    }
}
```

**Expected Output**

```
This is a Toyota vehicle.
Toyota engine is starting...
Toyota car is driving on the road.
Max speed: 200 km/h
```

**Why this output occurs**

- `Vehicle` provides the concrete `GetInfo` and virtual `StartEngine`; `Car` implements the abstract `Move` and `MaxSpeed`.

---

### Real-World Cases

- **Framework base classes** — `Stream`, `WebRequest`, `ControllerBase`.
- **Domain models** — `Entity`, `Shape`, `Vehicle`, `Account`.
- **Template Method pattern** — Abstract class defines the algorithm skeleton; derived classes fill in steps.

---

### References

- abstract keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- Abstract and sealed classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members
- Abstract Class vs Interface in C# — NDepend Blog - https://blog.ndepend.com/abstract-class-vs-interface-in-c/

---

## 2. `abstract` Keyword

### Definitions

**Core Definition**
The `abstract` keyword is a modifier that indicates its target has a missing or incomplete implementation. It is used on classes (to prevent instantiation) and on members (to require implementation in derived classes).

**Technical Definition**
The `abstract` modifier indicates that its target has a missing or incomplete implementation. It can be used with classes, methods, properties, indexers, and events. When used on a class, it indicates that the class is intended only to be a base class and not instantiated on its own. When used on a member, it indicates that the member has no implementation and must be overridden by non-abstract derived classes.

**Beginner-Friendly Explanation**
`abstract` is the **“you must fill this in”** marker. On a class, it means “do not create me directly.” On a member, it means “every child must provide this.”

---

### Purposes

- To prevent direct instantiation of a base class.
- To declare members that derived classes must implement.
- To enforce a contract on derived classes.
- To support polymorphic design through base-type references.

---

### Syntax Rules and Structure

```
abstract class ClassName { ... }          // abstract class
public abstract ReturnType Method();       // abstract method
public abstract ReturnType Property { get; }  // abstract property
```

**Component breakdown**

- `abstract` — The modifier.
- Class — Prevents instantiation.
- Member — No body; must be overridden.

#### Syntax Rules

- Abstract members have no body (just `;`).
- Abstract members are implicitly virtual.
- Abstract classes can contain abstract and non-abstract members.
- Non-abstract derived classes must implement all abstract members.

#### Constraints and Limitations

- Abstract members cannot be `static`, `virtual`, or `sealed`.
- Abstract classes cannot be `sealed`.
- Abstract members are permitted only in abstract classes (and interfaces).

---

### Annotated Code Examples and Expected Outputs

#### Example: `abstract` on Class and Members

```csharp
using System;

public abstract class Shape
{
    public abstract double Area();           // abstract method
    public abstract string Name { get; }     // abstract property
}

public class Circle : Shape
{
    public double Radius { get; set; }
    public override double Area() => Math.PI * Radius * Radius;
    public override string Name => "Circle";
}

class Program
{
    static void Main()
    {
        Shape s = new Circle { Radius = 2 };
        Console.WriteLine($"{s.Name}: {s.Area():F2}");   // Circle: 12.57
    }
}
```

**Expected Output**

```
Circle: 12.57
```

**Why this output occurs**

- `Shape` is abstract; `Circle` implements both abstract members.

---

### Real-World Cases

- **All abstract class declarations** — The keyword is mandatory.
- **Framework extension points** — Abstract members define required overrides.

---

### References

- abstract keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- Abstract and sealed classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members

---

## 3. Abstract Methods

### Definitions

**Core Definition**
An abstract method is a method declared without a body that must be implemented by every non-abstract derived class.

**Technical Definition**
When an instance method declaration includes the `abstract` modifier, the method is an abstract method. Abstract methods are implicitly virtual but cannot include the `virtual` modifier. They have no implementation (the body is just a semicolon) and are permitted only in abstract classes and interfaces. Non-abstract derived classes must override the method to provide an implementation. Abstract methods force derived classes to provide behavior, whereas virtual methods merely allow it.

**Beginner-Friendly Explanation**
An abstract method is a **blank to fill in**. It says “every child must provide this method.” If a child does not, that child must also be abstract.

---

### Purposes

- To force derived classes to implement specific behavior.
- To define a contract for a class hierarchy.
- To support polymorphic dispatch with guaranteed implementation.
- To model abstract concepts that have no meaningful default.

---

### Syntax Rules and Structure

```
public abstract ReturnType MethodName(parameters);
```

**Component breakdown**

- `abstract` — The modifier.
- No body (semicolon).

#### Syntax Rules

- Only in abstract classes or interfaces.
- Derived non-abstract classes must override.
- Cannot be `static`, `virtual`, `sealed`, or `extern`.
- Overrides use the `override` keyword.

#### Constraints and Limitations

- Cannot call `base.Method()` on an abstract method.
- Cannot be instantiated directly.

---

### Annotated Code Examples and Expected Outputs

#### Example: Abstract Method in a Shape Hierarchy

```csharp
using System;

public abstract class Shape
{
    public abstract double Area();
}

public class Square : Shape
{
    public double Side { get; set; }
    public override double Area() => Side * Side;
}

class Program
{
    static void Main()
    {
        Shape s = new Square { Side = 3 };
        Console.WriteLine(s.Area());   // 9
    }
}
```

**Expected Output**

```
9
```

**Why this output occurs**

- `Shape` declares `Area` abstract; `Square` must provide the implementation.

---

### Real-World Cases

- **Template Method pattern** — Abstract steps that derived classes implement.
- **Plugin contracts** — Abstract methods that plugins must implement.

---

### References

- abstract keyword — C# reference (abstract methods) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- Abstract and sealed classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members

---

## 4. Abstract Properties

### Definitions

**Core Definition**
An abstract property is a property declared without an implementation, which derived classes must implement by overriding it.

**Technical Definition**
Abstract properties behave like abstract methods, except for the differences in declaration and invocation syntax. An abstract property can declare `get`, `set`, or both accessors without providing bodies; derived classes must override the property and provide the accessor implementations. Abstract properties are permitted only in abstract classes (and interfaces, where they are implicitly abstract).

**Beginner-Friendly Explanation**
An abstract property is a **required data member**. The base class says “every child must have this property,” but each child decides how to implement it.

---

### Purposes

- To require derived classes to expose specific data.
- To enforce a property contract.
- To support polymorphic access to properties through base-type references.

---

### Syntax Rules and Structure

```
public abstract ReturnType PropertyName { get; set; }
public abstract ReturnType PropertyName { get; }
```

**Component breakdown**

- `abstract` — The modifier.
- `{ get; set; }` — Accessor declarations without bodies.

#### Syntax Rules

- Derived classes override with the `override` keyword.
- Can declare get-only, set-only, or read-write abstract properties.
- Abstract properties are implicitly virtual.

#### Constraints and Limitations

- Cannot have auto-implemented bodies.
- Must be overridden in non-abstract derived classes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Abstract Property with Getter and Setter

```csharp
using System;

public abstract class Document
{
    public abstract string Content { get; set; }
}

public class TextDocument : Document
{
    private string _content = "";
    public override string Content
    {
        get => _content;
        set => _content = value ?? "";
    }
}

class Program
{
    static void Main()
    {
        Document doc = new TextDocument();
        doc.Content = "Hello";
        Console.WriteLine(doc.Content);   // Hello
    }
}
```

**Expected Output**

```
Hello
```

**Why this output occurs**

- `Document` declares `Content` abstract; `TextDocument` provides the implementation.

---

### Real-World Cases

- **Domain entities** — Abstract `Id` or `Name` properties.
- **UI frameworks** — Abstract `Text` or `Value` properties on controls.

---

### References

- abstract keyword — C# reference (abstract properties) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- Abstract and sealed classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members

---

## 5. Concrete Methods

### Definitions

**Core Definition**
A concrete method (also called an implemented method) is a method with a full implementation that is inherited by derived classes and can be used as-is or optionally overridden.

**Technical Definition**
Concrete methods in an abstract class provide functionality that all derived classes share. They can be regular methods (non-virtual, not overridable), virtual methods (overridable with a default implementation), or methods with any access modifier. Abstract classes can contain fully implemented members such as regular methods, properties, and constructors, allowing them to provide common functionality while still requiring derived classes to implement specific abstract members. Concrete methods are inherited by derived classes and can be called on derived instances.

**Beginner-Friendly Explanation**
A concrete method is a **completed step** in the blueprint. The base class has already written the code, and every child inherits it.

---

### Purposes

- To share common functionality among derived classes.
- To avoid code duplication (DRY principle).
- To provide default behavior that derived classes can optionally override (if virtual).
- To offer utility methods that operate on the class’s state.

---

### Syntax Rules and Structure

```
// Regular concrete method (not overridable)
public ReturnType MethodName(parameters) { ... }

// Virtual concrete method (overridable)
public virtual ReturnType MethodName(parameters) { ... }
```

**Component breakdown**

- No `abstract` modifier.
- Body is provided.
- Can be `virtual` (overridable) or non-virtual (not overridable).

#### Syntax Rules

- Concrete methods can have any access modifier.
- Non-virtual concrete methods cannot be overridden (only hidden with `new`).
- Virtual concrete methods can be overridden with `override`.

#### Constraints and Limitations

- Non-virtual methods cannot participate in polymorphism.
- Virtual methods have a small dispatch cost.

---

### Annotated Code Examples and Expected Outputs

#### Example: Concrete Methods Shared by Derived Classes

```csharp
using System;

public abstract class Animal
{
    public string Name { get; set; }

    // Concrete method (non-virtual)
    public void Breathe() => Console.WriteLine($"{Name} is breathing.");

    // Virtual concrete method
    public virtual void Sleep() => Console.WriteLine($"{Name} is sleeping.");

    // Abstract method
    public abstract void MakeSound();
}

public class Dog : Animal
{
    public override void MakeSound() => Console.WriteLine($"{Name} says: Woof!");
    public override void Sleep() => Console.WriteLine($"{Name} is dreaming about bones.");
}

class Program
{
    static void Main()
    {
        Dog dog = new Dog { Name = "Rex" };
        dog.Breathe();   // Rex is breathing.
        dog.Sleep();     // Rex is dreaming about bones.
        dog.MakeSound(); // Rex says: Woof!
    }
}
```

**Expected Output**

```
Rex is breathing.
Rex is dreaming about bones.
Rex says: Woof!
```

**Why this output occurs**

- `Breathe` is inherited as-is; `Sleep` is overridden; `MakeSound` implements the abstract contract.

---

### Real-World Cases

- **Template Method** — Concrete methods define the algorithm skeleton; abstract/virtual steps are filled in by subclasses.
- **Base utility methods** — `ToString`, `Equals`, or domain-specific helpers.

---

### References

- Abstract and sealed classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members
- Compile-N-Run — .NET Abstract Classes (concrete methods) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/2-net-object-oriented-programming/10-net-abstract-classes.mdx

---

## 6. Abstract Class vs. Interface

### Definitions

**Core Definition**
Abstract classes and interfaces are both abstraction mechanisms, but they differ in **implementation sharing**, **inheritance semantics**, **state support**, **versioning flexibility**, and **value-type support**. Choosing between them is a fundamental design decision.

**Technical Definition**
Both abstract classes and interfaces define contracts that enforce method implementation in subclasses, but they differ in structure, capabilities, and intended use cases. An abstract class can contain both abstract and concrete members, fields, constructors, and supports single inheritance. An interface historically defines a pure contract with no implementation, supports multiple implementation, and cannot contain instance fields (though default implementations are allowed since C# 8). The Framework Design Guidelines recommend **favoring abstract classes over interfaces** because abstract classes version better: you can add a new virtual method without breaking derived classes, whereas adding a member to a shipped interface is a breaking change unless a default implementation is provided. However, interfaces are **required** when you need a common API supported by value types (structs cannot inherit from classes) or when a type must implement multiple contracts while already inheriting from another class.

**Beginner-Friendly Explanation**
An **abstract class** is a **partial blueprint**—it can have some completed parts and some parts you must fill in. An **interface** is a **pure contract**—it only says what must be done. Abstract classes are better when you want to share code; interfaces are better when you need maximum flexibility or need to work with structs.

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
| **Versioning** | Easier (add virtual/concrete members) | Harder (breaking change unless default methods) |
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

- Abstract Class vs Interface in C# — NDepend Blog - https://blog.ndepend.com/abstract-class-vs-interface-in-c/
- Abstract class vs Interface in our days — Microsoft Q&A - https://learn.microsoft.com/en-us/answers/questions/1131111/abstract-class-vs-interface-in-our-days
- If an abstract class does not have fields. Should it be made into an interface? — dotnet/runtime Discussion - https://github.com/dotnet/runtime/discussions/102616
- Abstract Class or Interface I — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/peaceofmind/abstract-class-or-interface-i

---

## 7. Abstract Class Best Practices

### Definitions

**Core Definition**
Abstract class best practices are the design guidelines that ensure abstract classes are **correctly constructed, safely extensible, and appropriately chosen** over interfaces or concrete classes.

**Technical Definition**
Best practices for abstract classes derive from the Framework Design Guidelines and community guidance. Key recommendations include: **do not define public or protected internal constructors in abstract types** (use `protected` or `internal` instead); **provide at least one concrete type that inherits from each abstract class you ship** (to validate the design); **favor abstract classes when you need to share code among closely related classes**; **avoid using abstract classes merely because two classes share some code** (prefer composition or helper methods); **maintain a true “is-a” relationship**; and **use abstract classes instead of interfaces to decouple the contract from implementations** when versioning is a priority.

**Beginner-Friendly Explanation**
Best practices keep abstract classes clean: use **protected constructors**, provide a **concrete child** to prove the design works, and only use abstract classes when there is a **genuine “is-a” relationship**—not just because two classes happen to share code.

---

### Purposes

- To ensure abstract classes are designed for extension.
- To validate the design with concrete implementations.
- To avoid misusing inheritance for code reuse.
- To improve maintainability and versioning.
- To align with .NET framework design conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Use protected constructors in abstract classes
public abstract class Vehicle
{
    protected Vehicle(string brand) { ... }
}

// Provide at least one concrete derived type
public class Car : Vehicle { ... }

// Favor abstract classes for shared code among related classes
// Avoid abstract classes for mere code sharing; prefer composition
// Maintain true "is-a" relationships
```

**Component breakdown**

- **Protected constructors** — Abstract types cannot be instantiated, so public constructors are misleading.
- **Concrete derived type** — Validates the abstract design and provides a usable type.
- **Shared code among related classes** — The primary use case for abstract classes.
- **Avoid code-reuse-only inheritance** — Prefer composition or helper methods.
- **True “is-a” relationship** — Abstract classes model genuine specialization.

#### Rules and Constraints

- **Do** use `protected` or `internal` constructors in abstract classes.
- **Do** provide at least one concrete type inheriting from each abstract class.
- **Do** favor abstract classes for shared code among related classes.
- **Do not** define `public` or `protected internal` constructors in abstract types.
- **Do not** use abstract classes solely for code reuse without an “is-a” relationship.
- **Do not** create deep abstract hierarchies unnecessarily.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Abstract Class

```csharp
using System;

public abstract class Shape
{
    protected Shape(string name) => Name = name;
    public string Name { get; }
    public abstract double Area();
    public virtual string Describe() => $"{Name}: {Area():F2}";
}

public sealed class Circle : Shape
{
    public double Radius { get; }
    public Circle(double radius) : base("Circle") => Radius = radius;
    public override double Area() => Math.PI * Radius * Radius;
}

class Program
{
    static void Main()
    {
        Shape s = new Circle(2);
        Console.WriteLine(s.Describe());   // Circle: 12.57
    }
}
```

**Expected Output**

```
Circle: 12.57
```

**Why this output occurs**

- `Shape` uses a `protected` constructor and provides a concrete `Circle` type, following best practices.

---

### Real-World Cases

- **Framework base classes** — `Stream`, `WebRequest` provide abstract designs with concrete implementations like `FileStream`, `HttpWebRequest`.
- **Domain abstractions** — `Entity<T>` with concrete `Customer`, `Order`.

---

### References

- Abstract Class Design — Microsoft Learn (Framework Design Guidelines) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/ms229047(v=vs.100)
- Abstract Class vs Interface in C# — NDepend Blog (best practices) - https://blog.ndepend.com/abstract-class-vs-interface-in-c/
- Compile-N-Run — .NET Abstract Classes (when to use) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/2-net-object-oriented-programming/10-net-abstract-classes.mdx
- If an abstract class does not have fields. Should it be made into an interface? — dotnet/runtime Discussion - https://github.com/dotnet/runtime/discussions/102616

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `abstract` classes | Core since C# 1.0; stable. |
| `abstract` members | Core since C# 1.0; stable. |
| Default interface methods | C# 8.0+; can make interfaces more abstract-class-like. |
| Static abstract interface members | C# 11+; enables generic math. |
| Abstract classes with `new()` constraint | **Not allowed** if the abstract class has abstract members. |
| Public constructors in abstract classes | **Discouraged** by Framework Design Guidelines. |

---

## References (Consolidated)

- abstract keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- Abstract and sealed classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members
- Abstract Class Design — Microsoft Learn (Framework Design Guidelines) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/ms229047(v=vs.100)
- Abstract Class vs Interface in C# — NDepend Blog - https://blog.ndepend.com/abstract-class-vs-interface-in-c/
- Abstract class vs Interface in our days — Microsoft Q&A - https://learn.microsoft.com/en-us/answers/questions/1131111/abstract-class-vs-interface-in-our-days
- If an abstract class does not have fields. Should it be made into an interface? — dotnet/runtime Discussion - https://github.com/dotnet/runtime/discussions/102616
- Compile-N-Run — .NET Abstract Classes - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/2-net-object-oriented-programming/10-net-abstract-classes.mdx

---