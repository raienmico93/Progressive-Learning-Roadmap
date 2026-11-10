# C# Sealed Classes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **sealed class** is a class that **cannot be inherited from**. The `sealed` modifier is applied to a class declaration to prevent other classes from using it as a base class. The same `sealed` modifier can also be applied to a method or property that **overrides** a `virtual` member, preventing further overriding in derived classes while still allowing the containing class to be inherited.

**Technical Definition**
When you apply the `sealed` modifier to a class, it prevents other classes from inheriting from that class. When applied to a method or property that overrides a `virtual` member, `sealed` must always be used together with `override`; it prevents a derived class from further overriding that specific member. It is a **compile-time error** to use the `abstract` modifier with a `sealed` class, because an abstract class must be inherited by a class that provides implementations. Structs are **implicitly sealed** and cannot be inherited. The `sealed` modifier does not affect the runtime behaviour of the type itself; it restricts the **inheritance graph** and enables certain JIT optimizations, most notably **devirtualization** of virtual calls on sealed types.

**Beginner-Friendly Explanation**
A sealed class is a **final class**—a class that says “this is the end of the line; no one can build on top of me.” Think of it as a **sealed envelope**: the contents are final and cannot be extended or modified by anyone else. You use `sealed` when a class is complete, well-tested, and not designed as an extension point. Sealing is also a performance tool: when the JIT compiler knows a class cannot be subclassed, it can often **skip the virtual method lookup** and call the method directly—sometimes making virtual calls dramatically faster. A **sealed method** is slightly different: it says “I am overriding a virtual method, but I am also locking it down so no one else can override it again.” Sealing is a deliberate design decision that communicates intent (“do not inherit from this”) and can improve performance, but it should be used thoughtfully—especially in public libraries, where sealing a class is a breaking change for anyone who was inheriting from it.

---

### Key Characteristics

- **No inheritance** — A sealed class cannot be used as a base class; attempting to derive from it produces compiler error **CS0509**.
- **Sealed methods require `override`** — The `sealed` modifier on a method or property must always be combined with `override`; it locks down an already-overridden virtual member.
- **Incompatible with `abstract`** — A sealed class cannot be abstract (CS0418), because abstract classes exist to be inherited.
- **Structs are implicitly sealed** — `struct` types cannot be inherited from; the `sealed` modifier is redundant on structs.
- **Enables JIT devirtualization** — Sealing a class allows the JIT compiler to devirtualize and inline virtual calls on instances of that type, because it knows no derived override can exist.
- **Communicates API intent** — Sealing signals that a type is **not an extension point** and should not be subclassed.
- **Breaking change for consumers** — In public libraries, sealing a previously unsealed class breaks any downstream code that derived from it.
- **Records can be sealed** — `public sealed record` is valid; records are classes and follow the same sealing rules.

---

### Prerequisites

- Understanding of C# classes, inheritance, and the `virtual`/`override` keywords.
- Familiarity with abstract classes and polymorphism.
- Knowledge of access modifiers and the inheritance graph.
- Awareness of the JIT compiler and virtual method dispatch (for the performance discussion).
- Basic understanding of API versioning and breaking changes (for best practices).

---

### Related Programming Areas

- **Object-oriented design** — Sealing is a design decision about extensibility and the inheritance contract.
- **Performance engineering** — Sealed classes enable JIT devirtualization, improving virtual call performance.
- **API design and versioning** — Sealing public types is a compatibility decision; the Framework Design Guidelines provide specific recommendations.
- **Records and immutability** — `sealed record` is a common pattern for immutable DTOs and value objects.
- **Struct design** — Structs are implicitly sealed; understanding this avoids redundant `sealed` modifiers.
- **Source generators** — Generated types are often sealed to prevent unintended extension.

---

### Core Concepts / Features

1. Sealed Classes
2. `sealed` Keyword
3. Sealed Methods
4. Sealed Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Sealed Classes

### Definitions

**Core Definition**
A sealed class is a class declared with the `sealed` modifier that cannot be used as a base class by any other class.

**Technical Definition**
When the `sealed` modifier is applied to a class declaration, the class is **closed for inheritance**. Any attempt to derive from a sealed class results in compiler error **CS0509** (“cannot derive from sealed type”). A sealed class can still inherit from other classes, implement interfaces, and contain virtual members—but those virtual members cannot be overridden by any derived class, because no derived class can exist. Sealed classes are **not** implicitly abstract; they can be instantiated normally. A sealed class cannot be declared `abstract` (CS0418). Structs are implicitly sealed, so the `sealed` modifier on a struct is redundant.

**Beginner-Friendly Explanation**
A sealed class is a **closed book**. You can read it (use it), but you cannot write a sequel (inherit from it). It is complete as-is.

---

### Purposes

- **To prevent unintended inheritance** and protect the class’s invariants.
- **To communicate API intent** that the type is not designed as an extension point.
- **To enable JIT devirtualization** and improve the performance of virtual calls on the type.
- **To lock down behaviour** in security-sensitive or framework-critical types.
- **To satisfy design guidelines** that recommend sealing leaf classes by default.

---

### Syntax Rules and Structure

```
[access modifier] sealed class ClassName [: baseClass] [, interfaces]
{
    // members
}
```

**Component breakdown**

- `sealed` — The modifier that closes the class to inheritance.
- `class` — The keyword introducing a reference type.
- `ClassName` — The class identifier.
- `: baseClass` — Optional base class (sealed classes can still inherit).
- `: interfaces` — Optional interface implementations.
- `{ ... }` — The class body.

#### Syntax Rules

- A sealed class cannot be used as a base class (CS0509).
- A sealed class cannot be `abstract` (CS0418).
- A sealed class can inherit from another class and implement interfaces.
- A sealed class can contain `virtual` members, but they cannot be overridden (since no derived class can exist).
- Structs are implicitly sealed; adding `sealed` to a struct is redundant.

#### Constraints and Limitations

- Sealing a public class is a **breaking change** for any consumer that was deriving from it.
- Sealed classes cannot be extended, so extension points must be provided through composition, interfaces, or virtual members on an unsealed base class.
- A sealed class cannot declare `new virtual` members (CS0549: “new virtual member in sealed class”).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sealed Class Preventing Inheritance

```csharp
using System;

public class A { }

public sealed class B : A { }   // B inherits from A, but B is sealed

// class C : B { }   // COMPILE ERROR CS0509: cannot derive from sealed type 'B'

class Program
{
    static void Main()
    {
        var b = new B();
        Console.WriteLine(b.GetType().Name);   // B
    }
}
```

**Expected Output**

```
B
```

**Why this output occurs**

- `B` is sealed, so no class can inherit from it. `A` remains inheritable, but `B` is a leaf in the inheritance graph.

---

#### Example 2: Sealed Class with Virtual Members

```csharp
using System;

public sealed class Calculator
{
    public virtual int Add(int a, int b) => a + b;
}

// class AdvancedCalculator : Calculator { }   // CS0509

class Program
{
    static void Main()
    {
        var calc = new Calculator();
        Console.WriteLine(calc.Add(3, 4));   // 7
    }
}
```

**Expected Output**

```
7
```

**Why this output occurs**

- The `virtual` method is allowed in a sealed class, but since no derived class can exist, the virtual modifier has no practical effect and the JIT can devirtualize the call.

---

#### Example 3: Sealed Record

```csharp
using System;

public sealed record Person(string FirstName, string LastName);

// public record Employee(string FirstName, string LastName, int Id)
//     : Person(FirstName, LastName);   // CS0509

class Program
{
    static void Main()
    {
        var p = new Person("Alice", "Smith");
        Console.WriteLine(p);   // Person { FirstName = Alice, LastName = Smith }
    }
}
```

**Expected Output**

```
Person { FirstName = Alice, LastName = Smith }
```

**Why this output occurs**

- `sealed record` prevents inheritance, which is common for DTOs and value objects that should not be extended.

---

### Real-World Cases

- **`string`** — The `System.String` class is sealed; you cannot inherit from it.
- **`decimal`** — The `System.Decimal` struct is implicitly sealed.
- **Framework utility types** — `System.Threading.Tasks.Task` is not sealed, but many framework types are sealed to protect invariants.
- **DTOs and value objects** — `sealed record` is a common pattern for immutable data carriers.

---

### References

- sealed modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed
- Abstract and Sealed Classes and Class Members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members
- sealed (C# Reference) — Microsoft Learn (English content, localized page) - https://learn.microsoft.com/fil-ph/dotnet/csharp/language-reference/keywords/sealed

---

## 2. `sealed` Keyword

### Definitions

**Core Definition**
The `sealed` keyword is a modifier that, when applied to a class, prevents inheritance from that class, and when applied to an overriding method or property, prevents further overriding of that member.

**Technical Definition**
The `sealed` modifier has two distinct uses in C#: on a **class**, it prevents other classes from inheriting from that class; on a **method or property that overrides a virtual member**, it prevents derived classes from further overriding that member. When applied to a method or property, `sealed` must always be used together with `override`. The `sealed` keyword is not allowed on interfaces, structs (which are implicitly sealed), or `abstract` classes.

**Beginner-Friendly Explanation**
`sealed` is the **“no more extending”** keyword. On a class, it means “this is final.” On an overriding method, it means “I override this, and nobody else can override it again.”

---

### Purposes

- **To close a class to inheritance** and make it a leaf in the type hierarchy.
- **To lock down a specific overridden member** while still allowing the containing class to be inherited.
- **To communicate design intent** that a type or member is not an extension point.
- **To enable performance optimizations** (devirtualization) by giving the JIT certainty about the runtime type.
- **To prevent accidental breaking changes** by disallowing subclassing of fragile types.

---

### Syntax Rules and Structure

```
// Sealed class
sealed class ClassName { ... }

// Sealed method or property (must be combined with override)
public sealed override ReturnType MethodName() { ... }
public sealed override ReturnType PropertyName { get; set; }
```

**Component breakdown**

- `sealed class` — Closes the class to inheritance.
- `sealed override` — Locks down an overridden member.
- `sealed` alone on a method or property is **invalid**; it must be combined with `override`.

#### Syntax Rules

- `sealed` on a class prevents inheritance (CS0509).
- `sealed` on a method or property must be combined with `override` (CS0238: “cannot be sealed because it is not an override”).
- `sealed` cannot be combined with `abstract` on a class (CS0418).
- `sealed` is redundant on structs (structs are implicitly sealed).
- `sealed` cannot be applied to interfaces.

#### Constraints and Limitations

- Sealing a public type is a breaking change for consumers.
- A sealed method must override a `virtual` or `abstract` member; it cannot seal a non-virtual member.
- A sealed class cannot contain `new virtual` members (CS0549).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `sealed` on a Class vs. `sealed` on a Method

```csharp
using System;

public class Base
{
    public virtual void Method1() => Console.WriteLine("Base.Method1");
    public virtual void Method2() => Console.WriteLine("Base.Method2");
}

public class Derived : Base
{
    // Sealed method: overrides Method1 and prevents further overriding
    public sealed override void Method1() => Console.WriteLine("Derived.Method1");

    // Non-sealed override: can be overridden further
    public override void Method2() => Console.WriteLine("Derived.Method2");
}

public class FurtherDerived : Derived
{
    // public override void Method1() { }   // CS0239: cannot override sealed member

    public override void Method2() => Console.WriteLine("FurtherDerived.Method2");
}

class Program
{
    static void Main()
    {
        Base b = new FurtherDerived();
        b.Method1();   // Derived.Method1 (sealed)
        b.Method2();   // FurtherDerived.Method2
    }
}
```

**Expected Output**

```
Derived.Method1
FurtherDerived.Method2
```

**Why this output occurs**

- `Method1` is sealed in `Derived`, so `FurtherDerived` cannot override it; the `Derived` implementation runs.
- `Method2` is not sealed, so `FurtherDerived` can override it.

---

#### Example 2: Sealed Class Cannot Be Abstract

```csharp
using System;

// public abstract sealed class Invalid { }   // CS0418: cannot be both abstract and sealed

public sealed class Valid
{
    public void DoWork() => Console.WriteLine("Working");
}

class Program
{
    static void Main()
    {
        new Valid().DoWork();   // Working
    }
}
```

**Expected Output**

```
Working
```

**Why this output occurs**

- A sealed class cannot be abstract; `Valid` is instantiable and complete.

---

### Real-World Cases

- **`sealed override` in framework types** — Locking down `ToString()` or `Equals()` in derived framework types.
- **Sealed DTOs** — `public sealed record` for immutable data transfer objects.
- **Security-critical types** — Sealing types that must not be extended to preserve invariants.

---

### References

- sealed modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed
- sealed (C# Reference) — Microsoft Learn - https://learn.microsoft.com/fil-ph/dotnet/csharp/language-reference/keywords/sealed

---

## 3. Sealed Methods

### Definitions

**Core Definition**
A **sealed method** is a method that overrides a `virtual` or `abstract` member from a base class and **prevents any further overriding** by derived classes. It is declared with both the `sealed` and `override` modifiers.

**Technical Definition**
A sealed method overrides an inherited virtual method with the same signature. A sealed method **shall also be marked with the `override` modifier**. Use of the `sealed` modifier prevents a derived class from further overriding the method. The method must have the same signature as the virtual method it overrides. A sealed method can be declared on a class that is itself not sealed—this allows the class to be inherited while locking down a specific member. The `sealed` modifier on a method **cannot** be used on a non-override method (CS0239). Sealed methods are a common pattern in framework design to prevent derived classes from changing critical behaviour while still allowing them to add new members.

**Beginner-Friendly Explanation**
A sealed method is a **locked override**. The base class said “you may override this.” The derived class said “I will override it, and I am locking it so no one else can override it again.”

---

### Purposes

- **To lock down a specific overridden behaviour** while still allowing the containing class to be inherited.
- **To prevent further customization** of a critical method in a class hierarchy.
- **To protect invariants** that must not be changed by deeper derived classes.
- **To enable the JIT to devirtualize calls** to that method on instances of the sealing class.
- **To express deliberate design intent** about which members are extension points and which are not.

---

### Syntax Rules and Structure

```
public sealed override ReturnType MethodName(parameters)
{
    // implementation
}
```

**Component breakdown**

- `public` — Accessibility (can be any access modifier).
- `sealed` — The modifier that prevents further overriding.
- `override` — **Required**; the method must override a virtual/abstract member.
- `ReturnType` — The method’s return type.
- `MethodName` — The method identifier.
- `parameters` — The parameter list (must match the overridden method).

#### Syntax Rules

- `sealed` must be combined with `override` (CS0239 otherwise).
- The method must have the same signature as the overridden virtual/abstract method.
- The containing class **need not** be sealed; only the method is sealed.
- A sealed method cannot be `abstract`, `virtual`, or `new`.
- A sealed method can be `async` if the overridden method is `async`.

#### Constraints and Limitations

- Cannot seal a non-override method.
- Cannot seal a `static` method.
- A sealed method cannot be overridden further (CS0239: “cannot override inherited member because it is sealed”).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sealed Method in an Unsealed Class

```csharp
using System;

public class X
{
    protected virtual void F() => Console.WriteLine("X.F");
    protected virtual void F2() => Console.WriteLine("X.F2");
}

public class Y : X
{
    // F is sealed: Z cannot override it
    protected sealed override void F() => Console.WriteLine("Y.F");

    // F2 is not sealed: Z can override it
    protected override void F2() => Console.WriteLine("Y.F2");
}

public class Z : Y
{
    // protected override void F() { }   // CS0239: F is sealed

    protected override void F2() => Console.WriteLine("Z.F2");
}

class Program
{
    static void Main()
    {
        var z = new Z();
        z.F2();   // Z.F2 (via protected access through a public wrapper is not shown)
    }
}
```

**Expected Output**

```
Z.F2
```

**Why this output occurs**

- `Y` seals `F`, so `Z` cannot override it; `F2` remains overridable, and `Z` overrides it.

---

#### Example 2: Sealed `ToString` Override

```csharp
using System;

public class Base
{
    public virtual string GetValue() => "Base";
}

public class Derived : Base
{
    public sealed override string GetValue() => "Derived";
}

public class Further : Derived
{
    // public override string GetValue() => "Further";   // CS0239
}

class Program
{
    static void Main()
    {
        Base b = new Further();
        Console.WriteLine(b.GetValue());   // Derived
    }
}
```

**Expected Output**

```
Derived
```

**Why this output occurs**

- `Derived` seals `GetValue`; `Further` cannot override it, so the `Derived` implementation is used even through a `Further` instance.

---

### Real-World Cases

- **Framework base classes** — Sealing `ToString()` or `Equals()` in intermediate base classes to lock down formatting or equality semantics.
- **Template Method pattern** — Sealing the algorithm skeleton while leaving individual steps virtual.
- **Security-sensitive hierarchies** — Sealing validation or authorization methods in a derived class so deeper subclasses cannot bypass them.

---

### References

- sealed modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed
- Classes — C# language specification (sealed methods) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- sealed (C# Reference) — Microsoft Learn - https://learn.microsoft.com/fil-ph/dotnet/csharp/language-reference/keywords/sealed

---

## 4. Sealed Best Practices

### Definitions

**Core Definition**
Sealed best practices are the design guidelines that help developers decide **when to seal** a class or method, balancing **extensibility**, **performance**, and **API compatibility**.

**Technical Definition**
Best practices for sealing derive from the Framework Design Guidelines and .NET performance guidance. Key recommendations include: **seal leaf classes by default** (classes that are not designed for inheritance and are not subclassed anywhere in the codebase); **do not seal classes that are intended as extension points**; **consider sealing public classes carefully**, because sealing a previously unsealed public class is a **breaking change**; **seal `record` types** when they are not intended for inheritance; **use `sealed override` to lock down critical members** while still allowing the class to be inherited; and **prefer composition over inheritance** when extensibility is needed but the base class should remain sealed. The performance case for sealing is strong: sealing enables **JIT devirtualization**, which can make virtual calls significantly faster and allows inlining and constant folding.

**Beginner-Friendly Explanation**
Best practices for sealing are about **balance**: seal when you are sure a class is not meant to be inherited, **do not seal** when you are providing an extension point, and **be careful with public classes** because sealing them can break other people’s code. Sealing is not just about performance—it is a **design statement** about how your type is meant to be used.

---

### Purposes

- **To provide a clear, consistent rule** for when to seal classes and methods.
- **To prevent breaking changes** in public APIs by avoiding unnecessary sealing of published types.
- **To maximize performance** by enabling devirtualization where appropriate.
- **To communicate extensibility intent** clearly through the type hierarchy.
- **To align with .NET framework design conventions** and performance guidance.

---

### Guidelines and Structure

#### General Guidelines

```
// DO: Seal leaf classes by default
public sealed class OrderProcessor
{
    public void Process(Order order) { /* ... */ }
}

// DO: Seal records not designed for inheritance
public sealed record OrderCreated(OrderId Id, CustomerId CustomerId);

// DO: Use sealed override to lock down critical members
public sealed override string ToString() => Value;

// DO NOT: Seal classes intended as extension points
public abstract class Shape   // abstract classes cannot be sealed
{
    public abstract double Area();
}

// DO NOT: Seal public types in published libraries without justification
// (breaking change for consumers who derive from them)

// PREFER: Composition over inheritance when extensibility is needed
public class Service
{
    private readonly IRepository _repo;   // has-a, not is-a
}
```

**Component breakdown**

- **Seal leaf classes by default** — Classes not subclassed anywhere should generally be sealed.
- **Seal records** — Immutable data records are typically sealed.
- **Sealed override** — Lock down critical members while allowing class inheritance.
- **Do not seal extension points** — Abstract classes and classes with virtual members intended for customization should not be sealed.
- **Public API caution** — Sealing a published unsealed class is a breaking change.
- **Composition over inheritance** — Prefer composition when you need flexibility without opening the base class to inheritance.

#### Rules and Constraints

- **Do** seal leaf classes by default (Cratis qualification: never seal public types of a published framework/library package without treating it as a semver-breaking change).
- **Do** seal records that are not intended for inheritance.
- **Do** use `sealed override` to lock down specific members.
- **Do not** seal classes designed for extension (abstract classes cannot be sealed anyway).
- **Do not** seal public classes in published libraries without a clear semver strategy.
- **Do not** use `sealed` on structs (they are implicitly sealed).

---

### Annotated Code Examples and Expected Outputs

#### Example: Sealed Leaf Class with Sealed Override

```csharp
using System;

public abstract class Shape
{
    public abstract double Area();
    public virtual string Describe() => $"Area = {Area():F2}";
}

public sealed class Circle : Shape
{
    public double Radius { get; }

    public Circle(double radius) => Radius = radius;

    public override double Area() => Math.PI * Radius * Radius;

    // Seal Describe so no further derived class (if one existed) could change it
    public sealed override string Describe() => $"Circle with area {Area():F2}";
}

// class ColoredCircle : Circle { }   // CS0509: Circle is sealed

class Program
{
    static void Main()
    {
        Shape s = new Circle(2);
        Console.WriteLine(s.Describe());   // Circle with area 12.57
    }
}
```

**Expected Output**

```
Circle with area 12.57
```

**Why this output occurs**

- `Circle` is a **sealed leaf class**; it implements the abstract `Area` and seals `Describe` to lock down the description format.

---

### Real-World Cases

- **Performance-sensitive libraries** — Sealing hot-path classes to enable devirtualization.
- **DTOs and value objects** — `sealed record` for immutable data carriers.
- **Framework extension points** — Leaving base classes unsealed and using `sealed override` to lock down critical members.

---

### References

- sealed modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed
- Framework Design Guidelines — Sealing classes - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/
- Seal Classes by Default (Cratis performance guidance) - https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/aaronontheweb/dotnet-skills/type-design-performance/SKILL.md
- Structural Patterns — Seal Classes for Devirtualization (Cratis) - https://github.com/cratis/applicationmodel/blob/ba5e6f4b4117960c7c81bab432287acb92ddf972/.cratis/ai/skills/analyzing-dotnet-performance/references/structural-patterns.md

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `sealed` on classes | Core since **C# 1.0**; stable. |
| `sealed override` on methods/properties | Core since **C# 1.0**; stable. |
| Structs are implicitly sealed | **Language rule**; adding `sealed` to a struct is redundant. |
| JIT devirtualization for sealed types | **Implementation-dependent**; .NET Core 3.0+ and .NET 6+ show significant improvements. |
| Sealing public types in published libraries | **Breaking change**; treat as a semver-major change. |
| `sealed` + `abstract` on a class | **Compile-time error** (CS0418). |
| `sealed` on a non-override method | **Compile-time error** (CS0239). |
| `new virtual` member in a sealed class | **Compile-time error** (CS0549). |

---

## References (Consolidated)

- sealed modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed
- Abstract and Sealed Classes and Class Members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members
- Classes — C# language specification (sealed methods) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- sealed (C# Reference) — Microsoft Learn (localized page with English content) - https://learn.microsoft.com/fil-ph/dotnet/csharp/language-reference/keywords/sealed
- Framework Design Guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/
- Seal Classes by Default (Cratis performance guidance) - https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/aaronontheweb/dotnet-skills/type-design-performance/SKILL.md
- Structural Patterns — Seal Classes for Devirtualization (Cratis) - https://github.com/cratis/applicationmodel/blob/ba5e6f4b4117960c7c81bab432287acb92ddf972/.cratis/ai/skills/analyzing-dotnet-performance/references/structural-patterns.md
- Performance Improvements in .NET 6 — .NET Blog (devirtualization) - https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-6/

---