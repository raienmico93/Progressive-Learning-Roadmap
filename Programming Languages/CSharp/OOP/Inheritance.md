# C# Inheritance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Inheritance is a fundamental object-oriented programming mechanism in C# that allows a class (the **derived class**) to **inherit**, **extend**, and **modify** the behaviour and data defined in another class (the **base class**). It enables the creation of class hierarchies, promotes code reuse, and supports polymorphism—the ability to treat objects of different derived types uniformly through a base-type reference.

**Technical Definition**
In C#, inheritance is expressed by placing a colon (`:`) after the derived class name, followed by the base class name. C# supports **single inheritance** for classes: a class can inherit directly from only one base class. However, a class can implement multiple interfaces. The base class’s members (except constructors and finalizers) are inherited by the derived class, and the derived class can access accessible inherited members, override `virtual` and `abstract` methods, and extend the base class with new members. The `sealed` modifier prevents a class from being used as a base class and prevents an overridden method from being overridden further. The `abstract` modifier forces derived classes to provide an implementation. **Composition over inheritance** is a design principle that recommends favouring “has-a” relationships (composition) over “is-a” relationships (inheritance) to reduce tight coupling and increase flexibility.

**Beginner-Friendly Explanation**
Inheritance is a **family tree for classes**. A **base class** (parent) defines common traits, and a **derived class** (child) inherits those traits and can add its own or change some of them. For example, an `Animal` base class might have a `Name` and a `Speak()` method; a `Dog` derived class inherits `Name` and can override `Speak()` to say “Woof!” Inheritance lets you write common code once and reuse it across related types. But inheritance is not always the right tool: **composition**—building a class from other objects rather than inheriting—is often more flexible because it avoids the tight coupling that inheritance creates.

---

### Key Characteristics

- **Single inheritance** — A C# class can inherit from exactly one base class.
- **Transitive inheritance** — If `C` inherits from `B` and `B` inherits from `A`, `C` inherits all accessible members of `A`.
- **Constructor chain** — Base-class constructors run before derived-class constructors; derived constructors use `base(...)` to pass arguments.
- **Polymorphism** — A base-type variable can hold a derived-type object; virtual dispatch selects the derived override at runtime.
- **`virtual` → `override` → `sealed override`** — A virtual method can be overridden; an overridden method can be sealed to prevent further overriding.
- **`abstract` forces implementation** — Abstract methods have no body and must be implemented by non-abstract derived classes.
- **`base` accesses the parent** — `base.Member` calls a base-class member; `base(...)` calls a base-class constructor.
- **Composition over inheritance** — Prefer “has-a” over “is-a” when flexibility and low coupling matter.

---

### Prerequisites

- Understanding of C# classes, objects, fields, properties, and methods.
- Familiarity with access modifiers (`public`, `protected`, `private`, `internal`).
- Knowledge of constructors and constructor chaining (`this`, `base`).
- Awareness of polymorphism and virtual method dispatch.
- Basic understanding of interfaces and abstraction.

---

### Related Programming Areas

- **Object-oriented programming** — Inheritance is one of the four pillars of OOP (alongside encapsulation, abstraction, and polymorphism).
- **Polymorphism** — Virtual and abstract methods enable polymorphic behaviour.
- **API design** — Base classes define contracts and extension points.
- **Composition** — The alternative to inheritance for building flexible object graphs.
- **Code reuse** — Inheritance is one mechanism for reuse, but composition is often preferred.

---

### Core Concepts / Features

1. Inheritance (Overarching Concept)
2. The Colon (`:`) Syntax
3. Base Class
4. Derived Class
5. Method Overriding (`virtual`, `override`, `sealed`, `abstract`)
6. `base` Keyword
7. Constructor Chaining
8. Inheritance Best Practices
9. Composition over Inheritance

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Inheritance (Overarching Concept)

### Definitions

**Core Definition**
Inheritance is the mechanism by which a class acquires the members of another class, enabling code reuse, hierarchical classification, and polymorphism.

**Technical Definition**
In C#, inheritance enables you to create new classes that reuse, extend, and modify the behaviour defined in other classes. The class whose members are inherited is called the **base class**, and the class that inherits those members is called the **derived class**. A derived class can have only one direct base class (single inheritance), but it can implement multiple interfaces. All members of the base class (except constructors and finalizers) are inherited by the derived class, though private members are accessible only within the base class itself.

**Beginner-Friendly Explanation**
Inheritance is **passing traits from parent to child**. The child class automatically gets the parent’s data and behaviour, and can add its own or modify inherited behaviour.

---

### Purposes

- To promote code reuse by sharing common members in a base class.
- To create logical class hierarchies that model “is-a” relationships.
- To enable polymorphism through virtual and abstract methods.
- To provide extensibility points for frameworks and libraries.
- To organise code into a maintainable, hierarchical structure.

---

### Syntax Rules and Structure

```
[access modifier] class DerivedClass : BaseClass
{
    // additional members and overrides
}
```

**Component breakdown**

- `DerivedClass` — The class that inherits.
- `:` — The inheritance operator.
- `BaseClass` — The class being inherited from.

#### Syntax Rules

- A class can inherit from exactly one base class.
- A class can implement multiple interfaces alongside the base class.
- The base class must be accessible to the derived class.
- Inheritance is transitive.

#### Constraints and Limitations

- Cannot inherit from a sealed class (CS0509).
- Cannot inherit from a struct.
- Constructors and finalizers are not inherited.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Inheritance

```csharp
using System;

public class Animal
{
    public string Name { get; set; }
    public void Eat() => Console.WriteLine($"{Name} is eating.");
}

public class Dog : Animal
{
    public void Bark() => Console.WriteLine($"{Name} says woof!");
}

class Program
{
    static void Main()
    {
        var dog = new Dog { Name = "Rex" };
        dog.Eat();   // Rex is eating.
        dog.Bark();  // Rex says woof!
    }
}
```

**Expected Output**

```
Rex is eating.
Rex says woof!
```

**Why this output occurs**

- `Dog` inherits `Name` and `Eat` from `Animal` and adds `Bark`.

---

#### Example 2: Transitive Inheritance

```csharp
using System;

public class A { public void MethodA() => Console.WriteLine("A"); }
public class B : A { public void MethodB() => Console.WriteLine("B"); }
public class C : B { public void MethodC() => Console.WriteLine("C"); }

class Program
{
    static void Main()
    {
        var c = new C();
        c.MethodA();   // A
        c.MethodB();   // B
        c.MethodC();   // C
    }
}
```

**Expected Output**

```
A
B
C
```

**Why this output occurs**

- `C` inherits from `B`, which inherits from `A`; all accessible methods are available on `C`.

---

### Real-World Cases

- **Framework hierarchies** — `Stream` → `FileStream`, `MemoryStream`.
- **UI frameworks** — `Control` → `Button`, `TextBox`.
- **Domain models** — `Entity` → `Customer`, `Order`.

---

### References

- Inheritance — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance
- Tutorial: Introduction to Inheritance — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/inheritance
- Inheritance — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#154-inheritance

---

## 2. The Colon (`:`) Syntax

### Definitions

**Core Definition**
The colon (`:`) is the syntactic operator that introduces a base class or interface list in a class declaration.

**Technical Definition**
In a class declaration, the colon follows the class name and precedes the base class and/or interface list. The base class must come first if both a base class and interfaces are specified. Interfaces are separated by commas. The colon is also used in constructor declarations to introduce a `base(...)` or `this(...)` initializer.

**Beginner-Friendly Explanation**
The colon means **“inherits from”** or **“implements.”** `Dog : Animal` reads “Dog inherits from Animal.”

---

### Purposes

- To specify the base class.
- To list implemented interfaces.
- To introduce constructor initializers (`base(...)`, `this(...)`).

---

### Syntax Rules and Structure

```
class Derived : Base, IInterface1, IInterface2
{
    public Derived() : base(args) { }
}
```

**Component breakdown**

- `: Base` — The base class.
- `: IInterface1, IInterface2` — Interface list.
- `: base(args)` — Base constructor call.
- `: this(args)` — Sibling constructor call.

#### Syntax Rules

- The base class must appear first in the list.
- Interfaces follow the base class, separated by commas.
- `base(...)` or `this(...)` must be the first part of a constructor.

#### Constraints and Limitations

- Cannot list multiple base classes.
- The base class must be accessible.

---

### Annotated Code Examples and Expected Outputs

#### Example: Base Class and Interfaces

```csharp
using System;

public interface IIdentifiable { int Id { get; } }

public class Entity { public int Id { get; set; } }

public class Customer : Entity, IIdentifiable
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

- `Customer` inherits `Id` from `Entity` and implements `IIdentifiable`.

---

### Real-World Cases

- **All inheritance declarations** — The colon is mandatory for inheritance.

---

### References

- Inheritance — C# Programming Guide (syntax) - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance
- Class declarations — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#152-class-declarations

---

## 3. Base Class

### Definitions

**Core Definition**
A base class is the class whose members are inherited by another class. It defines common data and behaviour shared by derived classes.

**Technical Definition**
The base class (sometimes called the parent or superclass) is the class being inherited from. It can be instantiated directly unless marked `abstract`. It can define `virtual` methods for derived classes to override, `abstract` methods that derived classes must implement, and non-virtual members that are inherited as-is. Private members are inherited but not accessible from derived classes. By default, any class can serve as a base class unless marked `sealed`.

**Beginner-Friendly Explanation**
The base class is the **parent** in the inheritance relationship. It holds the shared traits.

---

### Purposes

- To define common state and behaviour for a family of classes.
- To provide extension points via virtual and abstract methods.
- To establish a contract for derived classes.
- To enable polymorphic treatment of derived types.

---

### Syntax Rules and Structure

```
[access modifier] [abstract | sealed] class BaseClass
{
    // members
}
```

**Component breakdown**

- `abstract` — Prevents direct instantiation; may contain abstract members.
- `sealed` — Prevents inheritance.
- Members — Fields, properties, methods, etc.

#### Syntax Rules

- A base class can be `abstract` or concrete.
- A base class can be `sealed`, preventing inheritance.
- The base class's constructors are not inherited; derived classes call them via `base(...)`.

#### Constraints and Limitations

- A sealed base class cannot be inherited.
- Private base members are inaccessible to derived classes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Abstract Base Class

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

- `Shape` is an abstract base class; `Circle` implements `Area` and inherits `Describe`.

---

### Real-World Cases

- **Framework base classes** — `ControllerBase` in ASP.NET Core, `Stream`.
- **Domain abstractions** — `Entity`, `AggregateRoot`.

---

### References

- Base classes — Inheritance (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance
- abstract — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- sealed — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed

---

## 4. Derived Class

### Definitions

**Core Definition**
A derived class is a class that inherits members from a base class and can extend or override them.

**Technical Definition**
The derived class (sometimes called the child or subclass) inherits all accessible members of its base class, except constructors and finalizers. It can add new members, override `virtual` and `abstract` members, and access accessible base members through `base`. A derived class must call an accessible base constructor explicitly using `base(...)` or implicitly rely on an accessible parameterless base constructor. A derived class can be `sealed` to prevent further derivation.

**Beginner-Friendly Explanation**
The derived class is the **child**. It gets everything from the parent and can add or change things.

---

### Purposes

- To specialize base-class behaviour.
- To add new members specific to the derived type.
- To override virtual/abstract members.
- To participate in polymorphic hierarchies.

---

### Syntax Rules and Structure

```
[access modifier] [sealed] class DerivedClass : BaseClass
{
    // new members and overrides
}
```

**Component breakdown**

- `sealed` — Prevents further inheritance.
- `: BaseClass` — The base class.
- Members — New members and `override` members.

#### Syntax Rules

- Must call a base constructor (explicitly or implicitly).
- Can override virtual and abstract members.
- Can add new members.
- Can access `protected` and `public` base members.

#### Constraints and Limitations

- Cannot inherit from multiple base classes.
- Cannot access private base members.
- Cannot inherit from a sealed base class.

---

### Annotated Code Examples and Expected Outputs

#### Example: Derived Class with Override

```csharp
using System;

public class Animal
{
    public virtual void Speak() => Console.WriteLine("...");
}

public class Dog : Animal
{
    public override void Speak() => Console.WriteLine("Woof!");
}

class Program
{
    static void Main()
    {
        Animal a = new Dog();
        a.Speak();   // Woof!
    }
}
```

**Expected Output**

```
Woof!
```

**Why this output occurs**

- `Dog` overrides `Speak`; polymorphic dispatch calls the derived implementation.

---

### Real-World Cases

- **Specialized services** — `PremiumCustomerService : CustomerService`.
- **Custom exceptions** — `MyException : Exception`.

---

### References

- Derived classes — Inheritance (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance
- override — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override

---

## 5. Method Overriding (`virtual`, `override`, `sealed`, `abstract`)

### Definitions

**Core Definition**
Method overriding is the act of providing a new implementation for a `virtual` or `abstract` method inherited from a base class, using the `override` keyword. The modifiers `virtual`, `override`, `sealed`, and `abstract` control how methods participate in inheritance.

**Technical Definition**
By default, methods defined in a base class are **not overridable**. The `virtual` keyword allows derived classes to override a base class method. The `override` keyword is used in a derived class to implement a method that was declared `virtual` or `abstract` in the base class. An `abstract` method is implicitly virtual but has no body; non-abstract derived classes must override it. A `sealed override` method overrides an inherited virtual method but prevents further overriding by derived classes. The `sealed` modifier on a method must always be used with `override`.

**Beginner-Friendly Explanation**
- **`virtual`** — “You may override me.”
- **`override`** — “I am overriding my parent’s method.”
- **`abstract`** — “You must override me.”
- **`sealed override`** — “I override, and no one can override me again.”

---

### Purposes

- To enable polymorphism through runtime dispatch.
- To provide default behaviour that derived classes can customize (`virtual`).
- To enforce implementation in derived classes (`abstract`).
- To lock down behaviour in a derived class (`sealed override`).
- To support extensible frameworks and libraries.

---

### Syntax Rules and Structure

```
public virtual ReturnType Method() { ... }          // base: overridable
public override ReturnType Method() { ... }         // derived: override
public abstract ReturnType Method();                 // base: must override
public sealed override ReturnType Method() { ... }  // derived: no further override
```

**Component breakdown**

- `virtual` — Allows overriding.
- `override` — Replaces base implementation.
- `abstract` — No body; must be overridden.
- `sealed override` — Overrides but prevents further overriding.

#### Syntax Rules

- `override` requires a base method that is `virtual`, `abstract`, or `override`.
- `sealed` on a method requires `override`.
- `abstract` methods have no body (just `;`).
- An `abstract` class can override a `virtual` method with an `abstract` method, forcing reimplementation.

#### Constraints and Limitations

- Cannot override a non-virtual method.
- Cannot use `virtual` or `abstract` on `static` methods.
- `sealed override` cannot be further overridden.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `virtual` and `override`

```csharp
using System;

public class A
{
    public virtual void F() => Console.WriteLine("A.F");
}

public class B : A
{
    public override void F() => Console.WriteLine("B.F");
}

class Program
{
    static void Main()
    {
        A a = new B();
        a.F();   // B.F
    }
}
```

**Expected Output**

```
B.F
```

**Why this output occurs**

- `B` overrides the virtual `F`; runtime dispatch selects `B.F`.

---

#### Example 2: `abstract` Method

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

- `Shape` declares `Area` abstract; `Square` provides the implementation.

---

#### Example 3: `sealed override`

```csharp
using System;

public class A { public virtual void F() => Console.WriteLine("A.F"); }
public class B : A { public sealed override void F() => Console.WriteLine("B.F"); }
public class C : B
{
    // public override void F() { } // Compile error: sealed
}

class Program
{
    static void Main()
    {
        A a = new B();
        a.F();   // B.F
    }
}
```

**Expected Output**

```
B.F
```

**Why this output occurs**

- `B` seals its override; `C` cannot override `F`.

---

### Real-World Cases

- **Template method pattern** — Virtual methods define extension points.
- **Framework base classes** — `ControllerBase.OnActionExecuting` is virtual.
- **Custom exceptions** — Overriding `Message` or `ToString`.

---

### References

- virtual — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual
- override — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override
- abstract — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- sealed — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed
- Versioning with virtual, override, and new — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/versioning-with-the-override-and-new-keywords

---

## 6. `base` Keyword

### Definitions

**Core Definition**
The `base` keyword accesses members of the base class from within a derived class, and is also used to call a base-class constructor.

**Technical Definition**
The `base` keyword is used to access members of the base class from within a derived class. Use it to call a method on the base class that has been overridden by another method, or to specify which base-class constructor should be called when creating instances of the derived class. A base-class access is permitted only in a constructor, an instance method, or an instance property accessor. `base.Member` accesses a base member; `base(...)` calls a base constructor.

**Beginner-Friendly Explanation**
`base` means **“my parent.”** It lets a derived class call the parent’s version of a method or constructor.

---

### Purposes

- To call an overridden base method.
- To call a base-class constructor.
- To access base-class members hidden by derived members.
- To extend rather than replace base behaviour.

---

### Syntax Rules and Structure

```
base.Member
base(...)   // constructor call
```

**Component breakdown**

- `base.Member` — Accesses a base-class member.
- `base(...)` — Calls a base-class constructor.

#### Syntax Rules

- `base(...)` must be the first part of a constructor.
- `base` cannot be used in static members.
- The base member must be accessible (not private).

#### Constraints and Limitations

- Cannot access private base members.
- Cannot call an abstract base method via `base` (compile error).

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

class Program
{
    static void Main()
    {
        var c = new Circle(2.5);
        Console.WriteLine(c.Area().ToString("F2"));   // 19.63
    }
}
```

**Expected Output**

```
19.63
```

**Why this output occurs**

- `Circle` calls `base(radius, 0)` to initialise `Shape`.

---

### Real-World Cases

- **Template method pattern** — Derived classes call `base.Method()` to include base behaviour.
- **Constructor initialisation** — Passing required values to base constructors.

---

### References

- base keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base
- Instance constructors — C# Programming Guide (base keyword) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors

---

## 7. Constructor Chaining

### Definitions

**Core Definition**
Constructor chaining is the practice of one constructor calling another—either in the same class (`this(...)`) or in the base class (`base(...)`)`—to share initialization logic and ensure correct initialization order.

**Technical Definition**
When you declare an instance constructor in a derived class, you can call a constructor of the base class using the `base` keyword. The `base(...)` call must be the first thing the constructor does and can be used only once. Constructors execute in a chain starting with the least-derived class and ending with the most-derived class; the `Object` constructor always runs first. The `this(...)` keyword chains to a sibling constructor in the same class.

**Beginner-Friendly Explanation**
Constructor chaining is **calling one constructor from another**. `base(...)` says “run my parent’s constructor first”; `this(...)` says “run my sibling constructor first.”

---

### Purposes

- To pass required data to the base class.
- To avoid duplicating initialization logic.
- To ensure base-class state is initialized before derived-class state.
- To provide convenience overloads that delegate to a primary constructor.

---

### Syntax Rules and Structure

```
public Derived(args) : base(baseArgs) { ... }
public ClassName() : this(defaultArgs) { ... }
```

**Component breakdown**

- `: base(...)` — Calls a base constructor.
- `: this(...)` — Calls a sibling constructor.

#### Syntax Rules

- `base(...)` or `this(...)` must be the first part of the constructor.
- A constructor can chain to only one other constructor.
- If no `base(...)` is specified, the base parameterless constructor is called implicitly.

#### Constraints and Limitations

- Cannot chain to a constructor that does not exist or is inaccessible.
- Cannot chain to itself (cycle).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `base(...)` Chaining

```csharp
using System;

public class Animal
{
    public string Name { get; }
    public Animal(string name) => Name = name;
}

public class Dog : Animal
{
    public string Breed { get; }

    public Dog(string name, string breed) : base(name)
    {
        Breed = breed;
    }
}

class Program
{
    static void Main()
    {
        var d = new Dog("Rex", "Labrador");
        Console.WriteLine($"{d.Name}, {d.Breed}");   // Rex, Labrador
    }
}
```

**Expected Output**

```
Rex, Labrador
```

**Why this output occurs**

- `Dog` calls `base(name)` to initialize `Animal.Name`.

---

#### Example 2: `this(...)` Chaining

```csharp
using System;

public class Point
{
    public int X { get; }
    public int Y { get; }

    public Point() : this(0, 0) { }
    public Point(int x, int y) => (X, Y) = (x, y);
}

class Program
{
    static void Main()
    {
        var p = new Point();
        Console.WriteLine($"{p.X},{p.Y}");   // 0,0
    }
}
```

**Expected Output**

```
0,0
```

**Why this output occurs**

- The parameterless constructor delegates to `(0, 0)`.

---

### Real-World Cases

- **Framework base classes** — Derived classes pass data to base constructors.
- **Convenience overloads** — Short constructors delegate to full constructors.

---

### References

- Instance constructors — C# Programming Guide (base keyword) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors
- base keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base
- Constructor chaining — Stack Overflow (this vs. base) - https://stackoverflow.com/questions/3797528/how-to-do-constructor-chaining-in-c

---

## 8. Inheritance Best Practices

### Definitions

**Core Definition**
Inheritance best practices are the design guidelines that ensure inheritance is used **appropriately, safely, and maintainably**, and that class hierarchies remain flexible and well-documented.

**Technical Definition**
Best practices for inheritance derive from Microsoft’s Framework Design Guidelines and object-oriented design principles. Key recommendations include: **use inheritance only for true “is-a” relationships**; **prefer composition over inheritance** when flexibility is important; **make base classes abstract when they are not intended to be instantiated**; **prefer `virtual` methods over `new` for polymorphism**; **seal classes that are not designed for inheritance**; **document extension points**; and **avoid deep inheritance hierarchies**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep inheritance from causing problems: **only inherit when the child truly “is a” parent**, **keep hierarchies shallow**, **seal what should not be inherited**, and **document what derived classes can override**.

---

### Purposes

- To prevent misuse of inheritance (tight coupling, fragile hierarchies).
- To keep class hierarchies understandable and maintainable.
- To provide clear extension points.
- To reduce the risk of breaking changes.
- To align with .NET design conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Use inheritance for true "is-a" relationships
public abstract class Shape { ... }
public class Circle : Shape { ... }

// Prefer composition over inheritance when flexibility matters
public class Service
{
    private readonly IRepository _repo;   // has-a
    public Service(IRepository repo) => _repo = repo;
}

// Seal classes not designed for inheritance
public sealed class Utility { ... }

// Document extension points
/// <summary>Override to customize behavior.</summary>
public virtual void OnProcessing() { ... }
```

**Component breakdown**

- **“Is-a” relationships** — Inheritance models specialization.
- **Composition for flexibility** — “Has-a” avoids tight coupling.
- **Seal non-extensible classes** — Prevent unintended inheritance.
- **Document virtual methods** — Guide derived-class authors.
- **Shallow hierarchies** — Easier to understand and maintain.

#### Rules and Constraints

- **Do** use inheritance for true “is-a” relationships.
- **Do** prefer abstract base classes for frameworks and libraries.
- **Do** seal classes not designed for inheritance.
- **Do** document extension points.
- **Do not** use inheritance just for code reuse; prefer composition.
- **Do not** create deep hierarchies (more than 2–3 levels).
- **Do not** call virtual methods from constructors.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Hierarchy

```csharp
using System;

public abstract class Shape
{
    public abstract double Area();
}

public sealed class Circle : Shape
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
        Console.WriteLine(s.Area().ToString("F2"));   // 12.57
    }
}
```

**Expected Output**

```
12.57
```

**Why this output occurs**

- `Shape` is abstract (not instantiable); `Circle` is sealed (no further inheritance).

---

### Real-World Cases

- **Framework design** — Abstract base classes with well-documented virtual methods.
- **Application code** — Composition preferred for business logic.

---

### References

- Inheritance — C# Programming Guide (design guidance) - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance
- Framework Design Guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/

---

## 9. Composition over Inheritance

### Definitions

**Core Definition**
Composition over inheritance is a design principle that recommends building complex objects by **composing simpler, focused objects** (a “has-a” relationship) rather than inheriting from base classes (an “is-a” relationship). Composition provides greater flexibility and reduces tight coupling.

**Technical Definition**
Composition over inheritance is a design principle that favours building complex objects by composing simpler, focused objects rather than inheriting from base classes. This approach provides greater flexibility, reduces coupling, and follows the “has-a” relationship instead of forcing an “is-a” relationship where none logically exists. In C#, composition is typically implemented by injecting dependencies (interfaces or concrete objects) into a class via its constructor. Inheritance is acceptable primarily for framework integration points (e.g., `ControllerBase` in ASP.NET Core, custom exceptions inheriting from `Exception`) and when a genuine “is-a” relationship exists.

**Beginner-Friendly Explanation**
Composition over inheritance means **building objects from other objects instead of inheriting**. Instead of saying “a `Car` is an `Engine`,” you say “a `Car` **has an** `Engine`.” This makes it easy to swap the engine, test the car with a fake engine, and change behaviour without touching a class hierarchy.

---

### Purposes

- To reduce tight coupling between classes.
- To increase flexibility and testability.
- To avoid fragile base-class problems.
- To model “has-a” relationships naturally.
- To favour small, focused components over deep hierarchies.

---

### Syntax Rules and Structure (Composition Patterns)

```
// Composition: inject a collaborator
public class Service
{
    private readonly IRepository _repo;

    public Service(IRepository repo) => _repo = repo;

    public void DoWork() => _repo.Save();
}
```

**Component breakdown**

- **Collaborator** — The composed object (often an interface).
- **Constructor injection** — The collaborator is passed in.
- **Delegation** — The class calls the collaborator’s methods.

#### Syntax Rules

- Prefer interfaces for collaborators.
- Inject dependencies via the constructor.
- Delegate behaviour to the collaborator.
- Avoid exposing the collaborator directly unless needed.

#### Constraints and Limitations

- Composition can increase the number of objects.
- Some frameworks require inheritance (e.g., `ControllerBase`).
- Not all problems are best solved by composition; use judgement.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Inheritance (Bad for Flexibility)

```csharp
using System;

public class Engine
{
    public void Start() => Console.WriteLine("Engine started");
}

public class Car : Engine   // BAD: a car is not an engine
{
    public void Drive() => Start();
}

class Program
{
    static void Main()
    {
        var c = new Car();
        c.Drive();   // Engine started
    }
}
```

**Expected Output**

```
Engine started
```

**Why this is problematic**

- `Car` inherits from `Engine`, which is not a true “is-a” relationship; this tightly couples `Car` to `Engine`.

---

#### Example 2: Composition (Preferred)

```csharp
using System;

public interface IEngine { void Start(); }

public class Engine : IEngine
{
    public void Start() => Console.WriteLine("Engine started");
}

public class Car
{
    private readonly IEngine _engine;

    public Car(IEngine engine) => _engine = engine;

    public void Drive() => _engine.Start();
}

class Program
{
    static void Main()
    {
        var car = new Car(new Engine());
        car.Drive();   // Engine started
    }
}
```

**Expected Output**

```
Engine started
```

**Why this is better**

- `Car` **has an** `IEngine`; the engine can be swapped, mocked, or replaced without changing `Car`.

---

### Real-World Cases

- **Dependency injection** — Services compose repositories, loggers, and other services.
- **Strategy pattern** — Algorithms composed rather than inherited.
- **Testing** — Mocks and fakes injected via composition.

---

### References

- Composition over inheritance — Microsoft Q&A (recommendation) - https://learn.microsoft.com/en-gb/answers/questions/2156800/what-is-the-correct-way-to-call-a-method-that-has
- Composition over inheritance — Clean Code .NET - https://github.com/jeremymta/clean-code-dotnet
- Composition over inheritance — GitHub (design principle) - https://github.com/devnareshkumar/Composition-Over-Inheritance-CSharp

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Single class inheritance | **Language rule**; C# does not support multiple class inheritance. |
| `virtual` / `override` | Core since C# 1.0. |
| `sealed override` | Must combine `sealed` with `override`. |
| `abstract` methods | Only allowed in abstract classes and interfaces. |
| Calling virtual methods from constructors | **Discouraged**; derived overrides run before derived fields are initialized. |
| Composition over inheritance | **Design principle**, not a language rule; inheritance is acceptable for framework integration and true “is-a” relationships. |

---

## References (Consolidated)

- Inheritance — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance
- Tutorial: Introduction to Inheritance — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/inheritance
- Inheritance — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#154-inheritance
- virtual — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual
- override — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override
- abstract — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- sealed — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed
- base keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base
- Instance constructors — C# Programming Guide (base keyword) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors
- Versioning with virtual, override, and new — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/versioning-with-the-override-and-new-keywords
- Composition over inheritance — Microsoft Q&A - https://learn.microsoft.com/en-gb/answers/questions/2156800/what-is-the-correct-way-to-call-a-method-that-has
- Composition over inheritance — Clean Code .NET - https://github.com/jeremymta/clean-code-dotnet
- Composition over inheritance — GitHub (design principle) - https://github.com/devnareshkumar/Composition-Over-Inheritance-CSharp

---