# C# Polymorphism — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Polymorphism is a core principle of object-oriented programming that allows objects of different types to be treated uniformly through a common interface or base class. The word comes from Greek, meaning “many-shaped,” and it enables a single method call to behave differently depending on the actual runtime type of the object being operated on. In C#, polymorphism is expressed through method overloading, operator overloading, method overriding, and interface implementation.

**Technical Definition**
Polymorphism is often referred to as the third pillar of object-oriented programming, after encapsulation and inheritance. It has two distinct aspects: at **runtime**, objects of a derived class can be treated as objects of a base class in places such as method parameters and collections or arrays; the declared type of the object is no longer identical to its runtime type. Base classes may define and implement **virtual methods**, and derived classes can **override** them, providing their own definition and implementation. At **compile time**, polymorphism is achieved through **method overloading** (multiple methods with the same name but different parameter lists) and **operator overloading** (redefining operators for custom types). The C# language specification defines virtual, override, and abstract methods as the mechanisms for runtime polymorphism, while interfaces provide an alternative contract-based approach that enables polymorphic behavior without relying on class inheritance.

**Beginner-Friendly Explanation**
Polymorphism is the ability to **use the same command for different things**. Imagine a universal remote control with a “Play” button. Pressing “Play” starts a movie on a DVD player, plays a song on a music player, and starts a video game on a console. The button is the same, but the behavior depends on the device. In C#, a base class `Shape` might define a method `Draw()`. When you call `Draw()` on a `Circle` object, it draws a circle; on a `Square` object, it draws a square. You write the code once against the base type, and each derived type supplies its own behavior. This makes programs flexible and extensible: you can add new shapes without changing the code that draws them.

---

### Key Characteristics

- **Two main forms** — Compile-time (static) polymorphism via overloading, and runtime (dynamic) polymorphism via overriding and interfaces.
- **Virtual dispatch** — The runtime determines which override to invoke based on the object’s actual type, not the declared type of the reference.
- **Base-type references** — Derived objects can be assigned to base-type variables, parameters, and collections.
- **`virtual` → `override`** — Base classes declare virtual methods; derived classes override them with the `override` keyword.
- **`abstract` forces implementation** — Abstract methods have no body and must be implemented by non-abstract derived classes.
- **Interface-based polymorphism** — Interfaces define contracts that unrelated classes can implement, enabling polymorphism without a shared base class.
- **Design choice matters** — Prefer abstract classes when you control the hierarchy and need shared implementation; prefer interfaces when you need multiple inheritance of contracts or value-type support.

---

### Prerequisites

- Understanding of C# classes, methods, and inheritance.
- Familiarity with access modifiers (`public`, `protected`, `private`).
- Knowledge of virtual and abstract methods.
- Awareness of interfaces and their implementation.
- Basic understanding of the difference between compile-time and runtime binding.

---

### Related Programming Areas

- **Inheritance** — Polymorphism builds on class hierarchies; virtual methods are the primary runtime mechanism.
- **Interfaces** — Contract-based polymorphism enables decoupled, testable designs.
- **Design patterns** — Strategy, Factory, Template Method, and Visitor patterns rely heavily on polymorphism.
- **Dependency injection** — DI containers resolve dependencies polymorphically at runtime.
- **SOLID principles** — Polymorphism supports the Open/Closed Principle (open for extension, closed for modification).

---

### Core Concepts / Features

1. Polymorphism (Overarching Concept)
2. Compile-Time Polymorphism (Method Overloading, Operator Overloading)
3. Runtime Polymorphism (Method Overriding)
4. Virtual Methods
5. Abstract Methods
6. Interface Implementation
7. Polymorphism Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Polymorphism (Overarching Concept)

### Definitions

**Core Definition**
Polymorphism is the ability of a single interface or base-type reference to represent different underlying forms (data types), enabling code to operate uniformly on objects of different concrete types.

**Technical Definition**
Polymorphism is a Greek word meaning “many-shaped” and has two distinct aspects in C#: at runtime, objects of a derived class can be treated as objects of a base class in parameters, collections, and arrays, and virtual method dispatch selects the derived implementation based on the runtime type; at compile time, method overloading and operator overloading allow the same name or symbol to represent multiple operations, resolved by the compiler based on the arguments.

**Beginner-Friendly Explanation**
Polymorphism lets you **write code that works with many types through one common type**. Instead of writing separate code for circles, squares, and triangles, you write code for `Shape`, and each shape knows how to draw itself.

---

### Purposes

- To enable code reuse across related types through a common base class or interface.
- To make programs extensible: new types can be added without modifying existing code.
- To decouple callers from concrete implementations, improving testability and flexibility.
- To support design patterns such as Strategy, Factory, and Template Method.
- To simplify collections and method signatures by using base types.

---

### Syntax Rules and Structure

```
// Base-type reference holding a derived object
BaseType reference = new DerivedType();

// Virtual method declaration in base
public virtual ReturnType Method() { ... }

// Override in derived
public override ReturnType Method() { ... }
```

**Component breakdown**

- `BaseType reference` — A variable, parameter, or collection element typed as the base class or interface.
- `new DerivedType()` — The actual object; its runtime type determines virtual dispatch.
- `virtual` — Marks a base method as overridable.
- `override` — Replaces the base implementation in a derived class.

#### Syntax Rules

- A derived object can be assigned to a base-type variable (implicit upcast).
- Virtual dispatch occurs for virtual and abstract methods; non-virtual methods are bound at compile time.
- Interfaces provide an alternative polymorphic contract.

#### Constraints and Limitations

- Polymorphism through inheritance requires a shared base class.
- Overuse of polymorphism can introduce indirection and complexity.
- Not all problems benefit from polymorphic hierarchies; composition is often simpler.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Polymorphism with a Base Class

```csharp
using System;

public class Shape
{
    public virtual void Draw() => Console.WriteLine("Drawing a shape");
}

public class Circle : Shape
{
    public override void Draw() => Console.WriteLine("Drawing a circle");
}

public class Square : Shape
{
    public override void Draw() => Console.WriteLine("Drawing a square");
}

class Program
{
    static void Main()
    {
        Shape[] shapes = { new Circle(), new Square() };

        foreach (Shape s in shapes)
        {
            s.Draw();   // polymorphic call
        }
    }
}
```

**Expected Output**

```
Drawing a circle
Drawing a square
```

**Why this output occurs**

- Each `Shape` reference points to a different derived object; virtual dispatch calls the appropriate `Draw` override at runtime.

---

#### Example 2: Polymorphism with an Interface

```csharp
using System;

public interface ISound { void MakeSound(); }

public class Dog : ISound { public void MakeSound() => Console.WriteLine("Bark"); }
public class Cat : ISound { public void MakeSound() => Console.WriteLine("Meow"); }

class Program
{
    static void Main()
    {
        ISound[] sounds = { new Dog(), new Cat() };

        foreach (ISound s in sounds)
        {
            s.MakeSound();   // polymorphic call through the interface
        }
    }
}
```

**Expected Output**

```
Bark
Meow
```

**Why this output occurs**

- The interface `ISound` provides a common contract; each class implements it, and the runtime type determines which implementation runs.

---

### Real-World Cases

- **UI frameworks** — A `Control` array holds `Button`, `TextBox`, and `Label` objects; each renders itself polymorphically.
- **Payment processing** — An `IPaymentProcessor` interface is implemented by `StripeProcessor`, `PayPalProcessor`, etc., and the application processes payments uniformly.
- **Drawing applications** — A `List<Shape>` holds circles, squares, and triangles; each shape draws itself.

---

### References

- Polymorphism (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/polymorphism
- Polymorphism — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/introduction#virtual-override-and-abstract-methods

---

## 2. Compile-Time Polymorphism (Method Overloading, Operator Overloading)

### Definitions

**Core Definition**
Compile-time (static) polymorphism is the form of polymorphism resolved by the compiler at compile time, achieved through **method overloading** (multiple methods with the same name but different parameter lists) and **operator overloading** (redefining operators for custom types).

**Technical Definition**
Compile-time polymorphism is also known as **static binding** or **early binding**. The compiler determines which method or operator to invoke based on the number and types of the arguments provided at the call site. Method overloading allows multiple methods with the same name but different parameters in the same class; operator overloading allows custom types to define the behavior of operators such as `+`, `-`, `==`, and `!=`. The resolution is fixed at compile time and does not depend on runtime types.

**Beginner-Friendly Explanation**
Compile-time polymorphism is **“the compiler picks the right version before the program runs.”** You have several methods with the same name but different inputs, and the compiler figures out which one you mean based on what you pass.

---

### Purposes

- To provide convenient, intuitive APIs with the same method name for different input types.
- To allow custom types to use familiar operator syntax.
- To enable compile-time type checking and performance (no runtime dispatch overhead).
- To improve readability by avoiding arbitrary method name variations.

---

### Syntax Rules and Structure

```
// Method overloading
public ReturnType Method(int x) { ... }
public ReturnType Method(string x) { ... }

// Operator overloading
public static ReturnType operator +(Type a, Type b) { ... }
```

**Component breakdown**

- **Method overloading** — Same name, different parameter types or counts.
- **Operator overloading** — `public static` method using the `operator` keyword.

#### Syntax Rules

- Overloads must differ in parameter types or count (not just return type).
- Operators must be `public static` and at least one operand must be of the containing type.
- Overload resolution is performed at compile time based on the static types of the arguments.

#### Constraints and Limitations

- Cannot overload solely by return type.
- Some operators cannot be overloaded (`=`, `&&`, `||`, `[]`, `()`).
- Overloading can lead to ambiguous calls if not designed carefully.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Method Overloading

```csharp
using System;

public class Calculator
{
    public int Add(int a, int b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
    public double Add(double a, double b) => a + b;
}

class Program
{
    static void Main()
    {
        var calc = new Calculator();
        Console.WriteLine(calc.Add(5, 7));          // 12
        Console.WriteLine(calc.Add(5, 7, 3));       // 15
        Console.WriteLine(calc.Add(5.5, 7.5));      // 13
    }
}
```

**Expected Output**

```
12
15
13
```

**Why this output occurs**

- The compiler selects the overload whose parameter types match the arguments.

---

#### Example 2: Operator Overloading

```csharp
using System;

public readonly struct Complex
{
    public double Real { get; }
    public double Imaginary { get; }

    public Complex(double real, double imaginary)
    {
        Real = real;
        Imaginary = imaginary;
    }

    public static Complex operator +(Complex a, Complex b) =>
        new Complex(a.Real + b.Real, a.Imaginary + b.Imaginary);

    public override string ToString() => $"{Real} + {Imaginary}i";
}

class Program
{
    static void Main()
    {
        var a = new Complex(3, 4);
        var b = new Complex(2, 3);
        Console.WriteLine(a + b);   // 5 + 7i
    }
}
```

**Expected Output**

```
5 + 7i
```

**Why this output occurs**

- The overloaded `+` operator is resolved at compile time and adds the real and imaginary parts.

---

### Real-World Cases

- **`Console.WriteLine`** — Overloads for `int`, `string`, `double`, and `object`.
- **`Math.Max`** — Overloads for `int`, `long`, `double`, and `decimal`.
- **`DateTime`** — Overloaded `+` and `-` operators with `TimeSpan`.

---

### References

- Method overloading — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods
- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading

---

## 3. Runtime Polymorphism (Method Overriding)

### Definitions

**Core Definition**
Runtime (dynamic) polymorphism is the form of polymorphism resolved at runtime through **method overriding**, where a derived class provides a specific implementation of a method already defined in its base class.

**Technical Definition**
Runtime polymorphism is also known as **dynamic binding** or **late binding**. A derived class can override a base class member only if the base class member is declared as `virtual` or `abstract`. The derived member must use the `override` keyword to explicitly indicate that it participates in virtual dispatch. At runtime, when client code calls the method, the CLR looks up the object’s runtime type and invokes that override. This allows a base-type reference to invoke derived-class behavior based on the actual object type.

**Beginner-Friendly Explanation**
Runtime polymorphism is **“the object decides at runtime which method to run.”** You call a method through a base-type reference, and C# picks the version that matches the object’s actual type.

---

### Purposes

- To enable true polymorphic behavior in class hierarchies.
- To allow derived classes to customize inherited behavior.
- To support collections and parameters typed as the base class that work with any derived type.
- To enable extensible frameworks and plugin architectures.

---

### Syntax Rules and Structure

```
// Base class
public virtual ReturnType Method() { ... }

// Derived class
public override ReturnType Method() { ... }
```

**Component breakdown**

- `virtual` — Marks the base method as overridable.
- `override` — Provides a new implementation in the derived class.

#### Syntax Rules

- The base method must be `virtual`, `abstract`, or `override`.
- The derived method must use `override`.
- The signature must match exactly.

#### Constraints and Limitations

- Cannot override a non-virtual method.
- Calling virtual methods from constructors is discouraged (derived overrides run before derived fields are initialized).
- Overriding has a small performance cost due to virtual dispatch (usually negligible).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Overriding a Virtual Method

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

- The runtime type of `a` is `Dog`, so `Dog.Speak` is invoked.

---

#### Example 2: Polymorphic Array of Animals

```csharp
using System;

public class Animal { public virtual void Speak() => Console.WriteLine("..."); }
public class Dog : Animal { public override void Speak() => Console.WriteLine("Woof!"); }
public class Cat : Animal { public override void Speak() => Console.WriteLine("Meow!"); }

class Program
{
    static void Main()
    {
        Animal[] animals = { new Dog(), new Cat() };
        foreach (var a in animals)
            a.Speak();   // Woof! Meow!
    }
}
```

**Expected Output**

```
Woof!
Meow!
```

**Why this output occurs**

- Each element’s runtime type determines which override runs.

---

### Real-World Cases

- **UI event handling** — `Control.OnPaint` is overridden by custom controls.
- **Document processing** — `Document.Export` is overridden by `PdfDocument`, `WordDocument`.
- **Game engines** — `GameObject.Update` is overridden by player, enemy, and projectile classes.

---

### References

- Polymorphism (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/polymorphism
- override keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override

---

## 4. Virtual Methods

### Definitions

**Core Definition**
A virtual method is a method in a base class that provides a default implementation but can be **overridden** by derived classes to provide a specialized implementation.

**Technical Definition**
When an instance method declaration includes the `virtual` modifier, the method is a virtual method. A virtual method provides an implementation that derived classes can override using the `override` modifier. Only methods, properties, events, and indexers can be virtual; fields cannot be virtual. When a derived class overrides a virtual member, that member is called even when accessed through a base-type reference.

**Beginner-Friendly Explanation**
A virtual method is **“a default behavior that children can change.”** The parent provides a reasonable implementation, but each child can decide to do it differently.

---

### Purposes

- To provide a default implementation while allowing customization.
- To enable runtime polymorphism.
- To define extension points for derived classes.
- To support the Template Method design pattern.

---

### Syntax Rules and Structure

```
public virtual ReturnType MethodName(parameters)
{
    // default implementation
}
```

**Component breakdown**

- `virtual` — The modifier.
- A body is required (unlike abstract).

#### Syntax Rules

- Only instance methods, properties, events, and indexers can be virtual.
- Can be overridden with `override` or sealed with `sealed override`.
- Cannot be `static`, `abstract`, or `sealed` simultaneously with `virtual`.

#### Constraints and Limitations

- Virtual dispatch has a minor performance cost.
- Virtual methods cannot be private (they must be accessible to derived classes).

---

### Annotated Code Examples and Expected Outputs

#### Example: Virtual Method with Default Behavior

```csharp
using System;

public class Shape
{
    public virtual double Area() => 0;
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
        Console.WriteLine(s.Area().ToString("F2"));   // 12.57
    }
}
```

**Expected Output**

```
12.57
```

**Why this output occurs**

- `Shape.Area` provides a default (zero), but `Circle` overrides it with the correct formula.

---

### Real-World Cases

- **Framework base classes** — `Controller.OnActionExecuting` in ASP.NET Core.
- **Template Method pattern** — Base class defines the algorithm skeleton; virtual steps are overridden.

---

### References

- virtual keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual
- Polymorphism (C# Programming Guide) — Virtual members - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/polymorphism

---

## 5. Abstract Methods

### Definitions

**Core Definition**
An abstract method is a method declared without a body that **must be implemented** by every non-abstract derived class.

**Technical Definition**
When an instance method declaration includes the `abstract` modifier, the method is an abstract method. Abstract methods are implicitly virtual but cannot include the `virtual` modifier. They have no implementation (the body is just a semicolon) and are permitted only in abstract classes and interfaces. Non-abstract derived classes must override the method to provide an implementation. Abstract methods force derived classes to provide behavior, whereas virtual methods merely allow it.

**Beginner-Friendly Explanation**
An abstract method is a **contract with no implementation**. It says: “Every concrete child must provide this method.” It is like a job description that every hire must fulfill.

---

### Purposes

- To force derived classes to implement specific behavior.
- To define a contract for a class hierarchy.
- To support polymorphism with guaranteed implementation.
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

#### Constraints and Limitations

- Cannot be instantiated directly (abstract class).
- Cannot call `base.Method()` on an abstract method.

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

- **Framework base classes** — Abstract base classes with required overrides.
- **Plugin contracts** — Abstract methods that plugins must implement.

---

### References

- abstract keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- Polymorphism (C# Programming Guide) — Abstract members - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/polymorphism

---

## 6. Interface Implementation

### Definitions

**Core Definition**
Interface-based polymorphism allows classes to implement a common contract defined by an interface, enabling polymorphic behavior without relying on class inheritance.

**Technical Definition**
An interface defines a set of methods and properties that a class must implement, but does not provide implementation details. Classes can implement multiple interfaces, allowing them to define behavior for different aspects of the application. Interface-based polymorphism is demonstrated when objects of different unrelated classes are treated uniformly through a shared interface reference. This approach promotes loose coupling because callers depend on the interface rather than concrete implementations.

**Beginner-Friendly Explanation**
An interface is a **contract**. Any class that “signs” the contract must provide the listed methods. Polymorphism through interfaces means you can treat a `Dog`, a `Cat`, and even a `Doorbell` as `ISound` objects because they all implement the contract.

---

### Purposes

- To achieve polymorphism without a shared base class.
- To support multiple “inheritance” of contracts.
- To decouple callers from concrete implementations.
- To enable dependency injection and testability.
- To define behavior for value types (structs can implement interfaces).

---

### Syntax Rules and Structure

```
public interface IContract
{
    void Method();
}

public class Implementation : IContract
{
    public void Method() { ... }
}
```

**Component breakdown**

- `interface` — The contract declaration.
- `: IContract` — The implementing class.
- `public void Method()` — The implementation.

#### Syntax Rules

- A class can implement multiple interfaces.
- Interface members are implicitly public (pre-C# 8).
- Default interface members are allowed since C# 8.
- Interfaces can inherit from other interfaces.

#### Constraints and Limitations

- Cannot contain instance fields (static fields allowed in C# 8+).
- Cannot be instantiated directly.
- Default interface members require .NET Core 3.0+.

---

### Annotated Code Examples and Expected Outputs

#### Example: Interface-Based Polymorphism

```csharp
using System;

public interface ISound
{
    void MakeSound();
}

public class Dog : ISound
{
    public void MakeSound() => Console.WriteLine("The dog barks.");
}

public class Cat : ISound
{
    public void MakeSound() => Console.WriteLine("The cat meows.");
}

public class Doorbell : ISound
{
    public void MakeSound() => Console.WriteLine("The doorbell rings.");
}

class Program
{
    static void Main()
    {
        ISound[] sounds = { new Dog(), new Cat(), new Doorbell() };

        foreach (ISound s in sounds)
        {
            s.MakeSound();
        }
    }
}
```

**Expected Output**

```
The dog barks.
The cat meows.
The doorbell rings.
```

**Why this output occurs**

- Each class implements `ISound` with its own `MakeSound`; the interface reference dispatches polymorphically.

---

### Real-World Cases

- **Dependency injection** — `ILogger`, `IRepository` abstractions.
- **Plugin systems** — `IPlugin` contract for extensibility.
- **Testing** — Mocking interfaces with frameworks like Moq.

---

### References

- Interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/
- Interface-based polymorphism — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/implement-polymorphic-behavior/4-explore-interface-based-polymorphism

---

## 7. Polymorphism Best Practices

### Definitions

**Core Definition**
Polymorphism best practices are the design guidelines that ensure polymorphic hierarchies are **flexible, maintainable, and correctly chosen** between inheritance-based and interface-based approaches.

**Technical Definition**
Best practices for polymorphism derive from the Framework Design Guidelines and Microsoft’s polymorphic design considerations. Key recommendations include: **choose between inheritance-based and interface-based polymorphism based on the scenario**; **favour abstract classes when you control the hierarchy and need shared implementation**; **favour interfaces when you need multiple inheritance of contracts or support for value types**; **avoid overloading methods defined in base classes**; **do not add members without default implementations to published interfaces**; and **use extension methods when the interface is not under your control**.

**Beginner-Friendly Explanation**
Best practices help you **pick the right tool** for polymorphism: use abstract classes when you have a clear hierarchy and want to share code; use interfaces when you need flexibility, multiple contracts, or testability. Avoid changing published interfaces in breaking ways.

---

### Purposes

- To choose the appropriate polymorphic mechanism for each scenario.
- To keep hierarchies flexible and evolvable.
- To avoid breaking changes in published APIs.
- To improve testability and decoupling.
- To align with .NET design conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer abstract classes when you control the hierarchy
public abstract class Shape { public abstract double Area(); }

// Prefer interfaces for multiple contracts or value types
public interface IIdentifiable { int Id { get; } }

// Avoid overloading base-class methods
// Do not add members without defaults to published interfaces
// Use extension methods when the interface is not yours
```

**Component breakdown**

- **Inheritance-based** — Best for “is-a” relationships and shared implementation.
- **Interface-based** — Best for loose coupling, multiple contracts, and value types.
- **Avoid base-class overloading** — It provides no value and causes confusion.
- **Published interface stability** — Adding members without defaults breaks implementers.

#### Rules and Constraints

- **Do** choose inheritance for true “is-a” relationships with shared code.
- **Do** choose interfaces for decoupled contracts and multiple inheritance of contracts.
- **Do** use abstract classes to decouple contracts from implementations.
- **Do not** overload methods defined in base classes.
- **Do not** add members without default implementations to published interfaces.
- **Do not** use marker interfaces (interfaces with no members).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Choosing Inheritance vs. Interface

```csharp
using System;

// Inheritance: shared implementation
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

// Interface: contract without shared implementation
public interface IIdentifiable { int Id { get; } }

public class Entity : IIdentifiable
{
    public int Id { get; set; }
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

- `Shape` provides shared implementation (`Describe`); `IIdentifiable` provides a pure contract.

---

### Real-World Cases

- **Framework design** — Abstract classes for base frameworks; interfaces for cross-cutting contracts.
- **Application architecture** — Interfaces for repositories and services; abstract classes for shared domain logic.
- **API versioning** — Default interface members for evolving interfaces without breaking implementers.

---

### References

- Polymorphic design considerations — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/implement-polymorphic-behavior/8-summary
- Avoid overloading methods defined in base classes — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/member-overloading
- Default interface members — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#default-interface-members
- Abstract class vs. interface — Framework Design Guidelines (GitHub discussion) - https://github.com/dotnet/runtime/discussions/102616

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Default interface members | Introduced in **C# 8.0**; version-specific. |
| Static abstract interface members | Introduced in **C# 11**; version-specific. |
| Overloading base-class methods | **Discouraged**; causes confusion among callers. |
| Adding members to published interfaces without defaults | **Breaking change**; breaks all implementers. |
| Virtual dispatch performance | **Implementation-dependent**; the JIT optimises devirtualisation in some cases. |
| Calling virtual methods from constructors | **Discouraged**; derived overrides run before derived fields are initialized. |
| Marker interfaces | **Avoid**; use attributes instead. |

---

## References (Consolidated)

- Polymorphism (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/polymorphism
- Polymorphism — C# language specification (virtual, override, abstract) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/introduction#virtual-override-and-abstract-methods
- Method overloading — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods
- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading
- virtual keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual
- override keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override
- abstract keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract
- Interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/
- Interface-based polymorphism — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/implement-polymorphic-behavior/4-explore-interface-based-polymorphism
- Polymorphic design considerations — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/implement-polymorphic-behavior/8-summary
- Default interface members — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#default-interface-members
- Abstract class vs. interface — Framework Design Guidelines (GitHub discussion) - https://github.com/dotnet/runtime/discussions/102616
- Avoid overloading methods defined in base classes — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/member-overloading
- Polymorphism in .NET — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/2-net-object-oriented-programming/6-net-polymorphism.mdx

---