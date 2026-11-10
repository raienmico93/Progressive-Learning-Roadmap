# C# Constructors — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **constructor** is a special method of a class or struct that is invoked automatically when a new instance of the type is created. Its primary responsibility is to **initialize the new object** into a valid, usable state. Unlike ordinary methods, a constructor has no return type (not even `void`), its name must match the enclosing type, and it cannot be called explicitly after object creation—it is run by the runtime as part of the `new` expression. C# provides several constructor forms—default, parameterized, copy-like, static, private, and (since C# 12) primary constructors—each serving a distinct initialization role.

**Technical Definition**
A constructor is a member that initializes an instance of a class or struct. Constructors have the same name as the class or struct and no return type. Constructors can take parameters, can be overloaded, and can chain to other constructors using `this(...)` or to a base-class constructor using `base(...)`. If a class contains no explicit instance constructor, C# provides a **default parameterless constructor** that initializes fields to their default values. **Static constructors** (`static ClassName()`) run automatically before the type is first used and initialize static state; they cannot be called explicitly and have no parameters or access modifiers. **Private constructors** restrict instantiation and are commonly used with static factory methods or to prevent instantiation of a class containing only static members. **Primary constructors** (C# 12) allow constructor parameters to be declared directly on the type declaration; their parameters are in scope throughout the type body and are captured as needed. Constructors are not inherited; a derived class must invoke an accessible base constructor explicitly (via `base(...)`) or rely on an accessible parameterless base constructor.

**Beginner-Friendly Explanation**
A constructor is the **setup routine that runs automatically when you create an object**. Think of it as the “assembly instructions” that must be followed before an object is ready to use. When you write `new Person("Alice", 30)`, the constructor receives `"Alice"` and `30` and uses them to fill in the object’s fields. If you do not write any constructor, C# supplies a simple default one that leaves everything at zero/`null`/`false`. Some constructors are special: a **static constructor** runs once for the whole class (not per object), and a **private constructor** prevents outside code from creating instances at all—useful for utility classes and factory patterns. Since C# 12, a **primary constructor** lets you declare the constructor parameters right on the class header, reducing boilerplate.

---

### Key Characteristics

- **Same name as the type, no return type** — Constructors are identified by their name matching the enclosing type and by having no return type.
- **Automatic invocation** — Constructors run when `new` is evaluated; they cannot be called explicitly on an existing instance.
- **Not inherited** — Derived classes must invoke a base constructor explicitly with `base(...)` or rely on an accessible parameterless base constructor.
- **Overloadable** — A type can have multiple constructors with different parameter lists.
- **Chainable** — `this(...)` calls another constructor in the same type; `base(...)` calls a base-class constructor.
- **Default constructor** — Provided automatically only when no constructor is declared.
- **Static constructor** — Runs once per type, before first use; initializes static state.
- **Private constructor** — Prevents external instantiation; used for utility classes and factories.
- **Primary constructors (C# 12)** — Parameters declared on the type header; captured as fields when used.

---

### Prerequisites

- Understanding of C# classes, structs, fields, and properties.
- Familiarity with access modifiers (`public`, `private`, `protected`, `internal`).
- Knowledge of inheritance and base classes.
- Awareness of static members and type initialization.
- Basic understanding of object initializers and the `new` operator.

---

### Related Programming Areas

- **Object-oriented programming** — Constructors establish object invariants and dependencies.
- **Dependency injection** — Constructors accept services as dependencies.
- **Factory patterns** — Static factory methods combined with private constructors.
- **Immutable types** — Constructors assign `readonly`/`init` members.
- **Records and primary constructors** — Modern C# uses primary constructors extensively for data-centric types.

---

### Core Concepts / Features

1. Constructors (Overarching Concept)
2. Default Constructor
3. Parameterized Constructor
4. Copy Constructor (and `with` expressions)
5. Static Constructor
6. Private Constructor
7. Constructor Overloading
8. Constructor Chaining (`this()` and `base()`)
9. Primary Constructors (C# 12)
10. Object Initializers
11. Constructor Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Constructors (Overarching Concept)

### Definitions

**Core Definition**
A constructor is a special member that initializes a new instance of a class or struct.

**Technical Definition**
A constructor is a member that initializes an instance of a class or struct. Constructors have the same name as the class or struct and no return type. Constructors can take parameters and can be overloaded. Constructors are not inherited, but a derived class can invoke a base-class constructor using `base(...)`. Constructors can chain to other constructors in the same type using `this(...)`. If no instance constructor is declared, the compiler supplies a default parameterless constructor (for classes) that initializes fields to their default values.

**Beginner-Friendly Explanation**
A constructor is the **setup code that runs when you create an object**. It makes sure the object starts in a valid state.

---

### Purposes

- To initialize instance fields and properties.
- To enforce invariants at creation time.
- To acquire dependencies (e.g., services, configuration).
- To provide multiple ways to create an object (overloading).
- To share initialization logic via chaining.

---

### Syntax Rules and Structure

```
[access modifier] ClassName(parameters)
{
    // initialization
}
```

**Component breakdown**

- `access modifier` — `public`, `private`, `protected`, `internal`, etc.
- `ClassName` — Must match the enclosing type.
- `parameters` — Optional input values.
- Body — Initialization statements.

#### Syntax Rules

- Constructors have no return type (not even `void`).
- The constructor name must match the class or struct name.
- Constructors can be overloaded by parameter list.
- Constructors are not inherited.

#### Constraints and Limitations

- Constructors cannot be `static` (static constructors use a distinct syntax).
- Constructors cannot be `virtual`, `abstract`, `override`, or `sealed`.
- Constructors cannot be called explicitly on an existing object.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Constructor

```csharp
using System;

public class Person
{
    public string Name { get; }
    public int Age { get; }

    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

class Program
{
    static void Main()
    {
        var p = new Person("Alice", 30);
        Console.WriteLine($"{p.Name}, {p.Age}");   // Alice, 30
    }
}
```

**Expected Output**

```
Alice, 30
```

**Why this output occurs**

- `new Person("Alice", 30)` invokes the constructor, which assigns the parameters to the get-only properties.

---

#### Example 2: Constructor with Validation

```csharp
using System;

public class Account
{
    public decimal Balance { get; }

    public Account(decimal initialBalance)
    {
        if (initialBalance < 0)
            throw new ArgumentOutOfRangeException(nameof(initialBalance));
        Balance = initialBalance;
    }
}

class Program
{
    static void Main()
    {
        var acc = new Account(100m);
        Console.WriteLine(acc.Balance);   // 100
    }
}
```

**Expected Output**

```
100
```

**Why this output occurs**

- The constructor validates the input before assigning it to the read-only `Balance`.

---

### Real-World Cases

- **Entities** — Ensuring required data is present at creation.
- **Services** — Receiving injected dependencies via the constructor.
- **Value objects** — Validating and normalizing input.

---

### References

- Constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors
- Instance constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors
- Constructors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1511-constructors

---

## 2. Default Constructor

### Definitions

**Core Definition**
A default constructor is a parameterless constructor that takes no arguments. It is either declared explicitly or generated automatically by the compiler when no constructor is declared.

**Technical Definition**
If a class contains no instance constructor declarations, a **default instance constructor** is automatically provided. That default constructor simply invokes the parameterless constructor of the direct base class. If the class is abstract, the declared accessibility for the default constructor is `protected`; otherwise, it is `public`. If a class declares at least one instance constructor, the compiler does **not** provide a default constructor. For structs, the default constructor always exists and initializes all fields to default values; explicit parameterless struct constructors are allowed starting with C# 10.

**Beginner-Friendly Explanation**
A default constructor is a **no-argument constructor**. If you do not write any constructor, C# gives you one for free that sets everything to defaults.

---

### Purposes

- To allow object creation without arguments.
- To initialize fields to default values.
- To satisfy frameworks that require parameterless construction (e.g., serializers).
- To serve as a base for constructor chaining.

---

### Syntax Rules and Structure

```
public ClassName() { }
```

**Component breakdown**

- `public` — Accessibility (for non-abstract classes).
- `ClassName` — The class name.
- `()` — No parameters.

#### Syntax Rules

- Auto-provided only if no other constructor is declared.
- For abstract classes, the default constructor is `protected`.
- For structs, a default constructor always exists.

#### Constraints and Limitations

- If any constructor is declared, the implicit default constructor is not generated.
- The default constructor calls the base class's parameterless constructor.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Implicit Default Constructor

```csharp
using System;

public class Person
{
    public string Name;
    public int Age;
}

class Program
{
    static void Main()
    {
        var p = new Person();
        Console.WriteLine($"{p.Name ?? "null"}, {p.Age}");   // null, 0
    }
}
```

**Expected Output**

```
null, 0
```

**Why this output occurs**

- No constructor is declared, so the compiler provides a default one; fields receive default values.

---

#### Example 2: Explicit Default Constructor

```csharp
using System;

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public Person()
    {
        Name = "Unknown";
    }
}

class Program
{
    static void Main()
    {
        var p = new Person();
        Console.WriteLine(p.Name);   // Unknown
    }
}
```

**Expected Output**

```
Unknown
```

**Why this output occurs**

- The explicit parameterless constructor sets a meaningful default.

---

### Real-World Cases

- **Serialization** — Many serializers require a parameterless constructor.
- **ORM frameworks** — Entity materialization often uses parameterless constructors.
- **Structs** — Default construction is automatic.

---

### References

- Default constructors — Instance constructors (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors#parameterless-constructors
- Constructors — C# language specification (default constructors) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1511-constructors

---

## 3. Parameterized Constructor

### Definitions

**Core Definition**
A parameterized constructor is a constructor that accepts one or more arguments, allowing the caller to supply initial values or dependencies.

**Technical Definition**
A constructor can accept parameters to initialize the object’s state. Parameterized constructors are the primary way to pass required data or dependencies into a new object. They can be overloaded to provide multiple creation paths, and they can chain to other constructors using `this(...)` or to base-class constructors using `base(...)`.

**Beginner-Friendly Explanation**
A parameterized constructor is a **constructor that takes input**. It lets you say “create this object with these specific values.”

---

### Purposes

- To initialize objects with caller-supplied data.
- To enforce required dependencies.
- To provide multiple construction paths via overloading.
- To support immutable types with read-only fields.

---

### Syntax Rules and Structure

```
public ClassName(type1 param1, type2 param2, ...)
{
    // assign parameters to fields/properties
}
```

**Component breakdown**

- `public` — Accessibility.
- `ClassName` — The class name.
- `parameters` — One or more parameters.
- Body — Assignments and validation.

#### Syntax Rules

- Parameters can be of any type.
- The constructor can validate parameters before assigning.
- Can chain with `this(...)` or `base(...)`.

#### Constraints and Limitations

- Declaring any constructor suppresses the implicit default constructor.
- Parameter names should be meaningful and consistent.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Parameterized Constructor

```csharp
using System;

public class Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
}

class Program
{
    static void Main()
    {
        var p = new Point(3, 4);
        Console.WriteLine($"({p.X}, {p.Y})");   // (3, 4)
    }
}
```

**Expected Output**

```
(3, 4)
```

**Why this output occurs**

- The constructor assigns the parameters to the read-only properties.

---

### Real-World Cases

- **Dependency injection** — Services passed into constructors.
- **Value objects** — Required data enforced at creation.
- **Domain entities** — Identity and required fields set in constructor.

---

### References

- Parameterized constructors — Instance constructors (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors
- Constructors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1511-constructors

---

## 4. Copy Constructor (and `with` Expressions)

### Definitions

**Core Definition**
A copy constructor is a constructor that takes an instance of the same type and creates a copy of it. C# **does not provide** a built-in copy constructor; it must be written manually. For records, the compiler generates a **copy constructor** and a `with` expression for non-destructive mutation.

**Technical Definition**
A copy constructor is an instance constructor that takes a single parameter of the containing type and copies the state of that instance into the new one. C# does **not** implicitly generate copy constructors for classes or structs (except for `record` types, which receive a compiler-generated copy constructor and `with` expression support). To copy an object, you either write an explicit copy constructor or use `MemberwiseClone` (protected, shallow copy) or a `with` expression (for records and structs in C# 10+).

**Beginner-Friendly Explanation**
A copy constructor is a **constructor that copies another object**. C# does not give you one automatically for classes—you have to write it yourself. Records are the exception: they get copy support and the `with` expression.

---

### Purposes

- To create an independent copy of an object.
- To support cloning and snapshotting.
- To implement prototype patterns.
- To provide non-destructive mutation via `with` for records.

---

### Syntax Rules and Structure

```
public ClassName(ClassName other)
{
    // copy fields from other
}

// For records
var copy = original with { Property = newValue };
```

**Component breakdown**

- `ClassName other` — The source instance.
- Body — Copies fields/properties.
- `with` — Record/struct copy with modifications.

#### Syntax Rules

- Copy constructors are not generated for classes or structs.
- `record` types get a compiler-generated copy constructor and `with` support.
- C# 10+ allows `with` on structs and anonymous types.
- `MemberwiseClone` performs a shallow copy.

#### Constraints and Limitations

- Manual copy constructors require copying each field.
- `MemberwiseClone` is shallow; reference-type fields are shared.
- Deep copies require custom logic.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Manual Copy Constructor

```csharp
using System;

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }

    public Person(Person other)
    {
        Name = other.Name;
        Age = other.Age;
    }
}

class Program
{
    static void Main()
    {
        var p1 = new Person("Alice", 30);
        var p2 = new Person(p1);
        p2.Name = "Bob";
        Console.WriteLine(p1.Name);   // Alice
        Console.WriteLine(p2.Name);   // Bob
    }
}
```

**Expected Output**

```
Alice
Bob
```

**Why this output occurs**

- The copy constructor creates an independent copy; modifying `p2` does not affect `p1`.

---

#### Example 2: `with` Expression for Records

```csharp
using System;

public record Person(string Name, int Age);

class Program
{
    static void Main()
    {
        var p1 = new Person("Alice", 30);
        var p2 = p1 with { Age = 31 };
        Console.WriteLine(p1);   // Person { Name = Alice, Age = 30 }
        Console.WriteLine(p2);   // Person { Name = Alice, Age = 31 }
    }
}
```

**Expected Output**

```
Person { Name = Alice, Age = 30 }
Person { Name = Alice, Age = 31 }
```

**Why this output occurs**

- `with` creates a copy with the specified property changed; the original is unchanged.

---

### Real-World Cases

- **Prototype pattern** — Cloning configured objects.
- **Snapshotting** — Capturing state at a point in time.
- **Records** — Non-destructive updates via `with`.

---

### References

- Copy constructors — C# Programming Guide (records and `with`) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#positional-syntax-for-property-definition
- `with` expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression
- Object.MemberwiseClone — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.memberwiseclone

---

## 5. Static Constructor

### Definitions

**Core Definition**
A static constructor is a special constructor that runs **once per type**, before the type is first used, to initialize static state.

**Technical Definition**
A static constructor is used to initialize any static data, or to perform a particular action that needs to be performed only once. It is called automatically before the first instance is created or any static members are referenced. A static constructor has the form `static ClassName()`. It takes no parameters, has no access modifiers, and cannot be called explicitly. If a static constructor throws an exception, the type becomes unusable for the lifetime of the application domain (`TypeInitializationException`).

**Beginner-Friendly Explanation**
A static constructor is the **class’s own setup routine**. It runs once—before anything else uses the class—and sets up shared data.

---

### Purposes

- To initialize static fields with non-trivial logic.
- To read configuration or environment data once.
- To enforce invariants on static state.
- To perform one-time setup for the type.

---

### Syntax Rules and Structure

```
static ClassName()
{
    // static initialization
}
```

**Component breakdown**

- `static` — The modifier.
- `ClassName` — The type name.
- `()` — No parameters.

#### Syntax Rules

- No access modifiers, no parameters.
- Runs once per application domain.
- Runs before first instance creation or static member access.
- Cannot be called explicitly.

#### Constraints and Limitations

- Cannot be overloaded.
- Cannot be called directly.
- Exceptions in a static constructor render the type unusable.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Static Constructor Initializing a Field

```csharp
using System;

public class Config
{
    public static readonly string AppName;

    static Config()
    {
        AppName = "MyApp";
        Console.WriteLine("Static constructor ran");
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine(Config.AppName);   // Static constructor ran
                                              // MyApp
        Console.WriteLine(Config.AppName);   // MyApp (no rerun)
    }
}
```

**Expected Output**

```
Static constructor ran
MyApp
MyApp
```

**Why this output occurs**

- The static constructor runs once, before the first access to `AppName`.

---

#### Example 2: Static Constructor Exception

```csharp
using System;

public class Broken
{
    public static int Value;

    static Broken()
    {
        throw new InvalidOperationException("Init failed");
    }
}

class Program
{
    static void Main()
    {
        try
        {
            Console.WriteLine(Broken.Value);
        }
        catch (TypeInitializationException ex)
        {
            Console.WriteLine(ex.InnerException?.Message);   // Init failed
        }
    }
}
```

**Expected Output**

```
Init failed
```

**Why this output occurs**

- The exception thrown in the static constructor is wrapped in `TypeInitializationException`.

---

### Real-World Cases

- **Configuration** — Reading environment variables once.
- **Singleton instances** — Lazy or eager static initialization.
- **Caches** — Precomputing static lookup tables.

---

### References

- Static constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors
- Static constructors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1514-static-constructors

---

## 6. Private Constructor

### Definitions

**Core Definition**
A private constructor is a constructor declared with `private` accessibility, preventing external code from creating instances of the class.

**Technical Definition**
A private constructor is a special instance constructor used in classes that contain only static members, or to prevent instantiation of a class from outside the class itself. Private constructors are commonly used with static factory methods (e.g., `Guid.NewGuid`) and to implement the singleton pattern. A class with only a private constructor cannot be instantiated by external code, and cannot be inherited from unless the nested class has access to the private constructor.

**Beginner-Friendly Explanation**
A private constructor is a **locked door**. Only code inside the class (or nested classes) can create instances. It is used when you want to control creation—through a factory method or a singleton.

---

### Purposes

- To prevent external instantiation.
- To implement the singleton pattern.
- To implement static factory methods.
- To group static utility members in a class that cannot be instantiated.

---

### Syntax Rules and Structure

```
private ClassName() { }
```

**Component breakdown**

- `private` — The accessibility.
- `ClassName` — The class name.
- `()` — Parameters (optional).

#### Syntax Rules

- The constructor is accessible only within the class.
- External code cannot use `new ClassName()`.
- Can be combined with public static factory methods.

#### Constraints and Limitations

- Cannot be instantiated externally.
- Inheritance is restricted unless the base provides an accessible constructor.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Private Constructor with Static Factory

```csharp
using System;

public class Temperature
{
    public double Celsius { get; }

    private Temperature(double celsius) => Celsius = celsius;

    public static Temperature FromCelsius(double c) => new Temperature(c);
    public static Temperature FromFahrenheit(double f) => new Temperature((f - 32) * 5 / 9);
}

class Program
{
    static void Main()
    {
        var t = Temperature.FromFahrenheit(212);
        Console.WriteLine(t.Celsius);   // 100
    }
}
```

**Expected Output**

```
100
```

**Why this output occurs**

- The constructor is private; callers use the static factory methods.

---

#### Example 2: Singleton Pattern

```csharp
using System;

public sealed class Singleton
{
    private static readonly Singleton _instance = new Singleton();

    public static Singleton Instance => _instance;

    private Singleton() { Console.WriteLine("Created"); }

    public void DoWork() => Console.WriteLine("Working");
}

class Program
{
    static void Main()
    {
        var s = Singleton.Instance;
        s.DoWork();   // Working
    }
}
```

**Expected Output**

```
Created
Working
```

**Why this output occurs**

- The private constructor ensures only one instance is created by the class itself.

---

### Real-World Cases

- **Singletons** — One instance per process.
- **Factory methods** — Controlled creation with validation or caching.
- **Utility classes** — Prevent instantiation of static-only classes.

---

### References

- Private constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/private-constructors
- Private constructors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1511-constructors

---

## 7. Constructor Overloading

### Definitions

**Core Definition**
Constructor overloading is the declaration of multiple constructors with different parameter lists in the same type.

**Technical Definition**
A type can declare multiple instance constructors, provided each has a unique signature (parameter types and count). Overloaded constructors allow objects to be created in different ways. Constructors can chain to one another using `this(...)` to share initialization logic and avoid duplication.

**Beginner-Friendly Explanation**
Constructor overloading is having **several ways to build the same object**. One constructor might take a name and age; another might take just a name and use a default age.

---

### Purposes

- To provide multiple creation paths.
- To supply default values.
- To support different input formats.
- To share logic via chaining.

---

### Syntax Rules and Structure

```
public ClassName() { ... }
public ClassName(string name) { ... }
public ClassName(string name, int age) { ... }
```

**Component breakdown**

- Multiple constructors with distinct parameter lists.

#### Syntax Rules

- Constructors must differ in parameter types or count.
- Constructors can chain with `this(...)`.

#### Constraints and Limitations

- Duplicate signatures cause compile errors.
- Declaring any constructor suppresses the implicit default constructor.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Overloaded Constructors

```csharp
using System;

public class Person
{
    public string Name { get; }
    public int Age { get; }

    public Person() : this("Unknown", 0) { }
    public Person(string name) : this(name, 0) { }
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine(new Person().Name);            // Unknown
        Console.WriteLine(new Person("Alice").Age);      // 0
        Console.WriteLine(new Person("Bob", 30).Age);    // 30
    }
}
```

**Expected Output**

```
Unknown
0
30
```

**Why this output occurs**

- Each overload delegates to the most detailed constructor via `this(...)`.

---

### Real-World Cases

- **Configuration objects** — Defaults vs. explicit values.
- **DTOs** — Convenience constructors.

---

### References

- Constructor overloading — Constructors (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors
- Instance constructors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1511-constructors

---

## 8. Constructor Chaining (`this()` and `base()`)

### Definitions

**Core Definition**
Constructor chaining is the practice of having one constructor call another—either in the same class (`this(...)`) or in the base class (`base(...)`)`—to share initialization logic.

**Technical Definition**
A constructor can use `this(...)` to invoke another constructor in the same class, or `base(...)` to invoke a base-class constructor. The `this(...)` or `base(...)` call must be the first statement of the constructor. Every instance constructor implicitly calls a base-class parameterless constructor unless it explicitly chains with `base(...)`. Constructor chaining ensures that shared initialization logic is written once and executed in the correct order.

**Beginner-Friendly Explanation**
Constructor chaining is **calling one constructor from another**. `this(...)` says “run my sibling constructor first”; `base(...)` says “run my parent’s constructor first.”

---

### Purposes

- To avoid duplicating initialization logic.
- To provide convenience overloads that delegate to a primary constructor.
- To ensure base-class initialization happens correctly.
- To centralize validation and assignment.

---

### Syntax Rules and Structure

```
public ClassName() : this(defaultArgs) { }
public ClassName(args) : base(baseArgs) { ... }
```

**Component breakdown**

- `: this(...)` — Chains to a sibling constructor.
- `: base(...)` — Chains to a base constructor.

#### Syntax Rules

- The chain call must be the first part of the constructor.
- A constructor can chain to only one other constructor.
- If no `base(...)` is specified, the base parameterless constructor is called implicitly.

#### Constraints and Limitations

- Cannot chain to itself (cycle).
- The chained constructor must exist and be accessible.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `this(...)` Chaining

```csharp
using System;

public class Point
{
    public int X { get; }
    public int Y { get; }

    public Point() : this(0, 0) { }
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
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

#### Example 2: `base(...)` Chaining

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

### Real-World Cases

- **Framework base classes** — Derived classes pass data to base constructors.
- **Convenience overloads** — Short constructors delegate to full constructors.

---

### References

- this keyword — C# reference (constructor chaining) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/this
- base keyword — C# reference (constructor calls) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base
- Instance constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors

---

## 9. Primary Constructors (C# 12)

### Definitions

**Core Definition**
A primary constructor (C# 12) is a constructor whose parameters are declared directly on the type declaration, making them available throughout the type body.

**Technical Definition**
Primary constructors are a feature introduced in C# 12. You can declare a constructor with parameters directly on a class, struct, or record. These parameters are in scope throughout the type body and are captured as fields when used. They are commonly used with records, but are also available for classes and structs. Primary constructor parameters are not stored as fields unless they are used in the type body; the compiler captures them in a hidden field only when needed. For classes, the primary constructor runs as part of object initialization; for structs, the primary constructor must initialize all fields.

**Beginner-Friendly Explanation**
A primary constructor lets you **declare constructor parameters on the class header**: `public class Person(string name, int age) { ... }`. You do not need a separate constructor block; the parameters are available throughout the class.

---

### Purposes

- To reduce boilerplate for classes with simple initialization.
- To make constructor parameters part of the type declaration.
- To simplify data-centric types (especially records).
- To avoid explicitly declaring fields for constructor parameters.

---

### Syntax Rules and Structure

```
public class ClassName(type1 param1, type2 param2)
{
    // parameters are in scope here
}
```

**Component breakdown**

- `(type1 param1, ...)` — Primary constructor parameters on the type header.
- Body — Members can reference the parameters.

#### Syntax Rules

- Available in C# 12+.
- Parameters are in scope throughout the type body.
- Parameters are captured as fields only when used.
- Records use primary constructor parameters to generate properties.

#### Constraints and Limitations

- Primary constructor parameters are not automatically exposed as properties on classes (unlike records).
- The parameters are captured in a hidden field if used; this can affect memory layout.
- Primary constructors cannot be combined with explicit parameterless constructors in the same way as traditional constructors.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Primary Constructor on a Class

```csharp
using System;

public class Person(string name, int age)
{
    public string Name { get; } = name;
    public int Age { get; } = age;

    public string Describe() => $"{Name} is {Age}";
}

class Program
{
    static void Main()
    {
        var p = new Person("Alice", 30);
        Console.WriteLine(p.Describe());   // Alice is 30
    }
}
```

**Expected Output**

```
Alice is 30
```

**Why this output occurs**

- The primary constructor parameters `name` and `age` are used to initialize the properties.

---

#### Example 2: Primary Constructor on a Struct

```csharp
using System;

public struct Point(int x, int y)
{
    public int X { get; } = x;
    public int Y { get; } = y;

    public override string ToString() => $"({X}, {Y})";
}

class Program
{
    static void Main()
    {
        var p = new Point(1, 2);
        Console.WriteLine(p);   // (1, 2)
    }
}
```

**Expected Output**

```
(1, 2)
```

**Why this output occurs**

- The primary constructor parameters initialize the struct’s properties.

---

### Real-World Cases

- **Records** — The default way to define positional records.
- **DTOs and value objects** — Concise initialization.
- **Service classes** — Constructor parameter capture for dependencies.

---

### References

- Primary constructors — C# 12 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12#primary-constructors
- Primary constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors#primary-constructors
- Primary constructors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-12.0/primary-constructors

---

## 10. Object Initializers

### Definitions

**Core Definition**
An object initializer is syntax that creates an object and assigns values to its properties or fields in a single expression, without requiring a constructor call for each value.

**Technical Definition**
Object initializers let you assign values to any accessible fields or properties of an object when you create it, without having to invoke a constructor explicitly. The syntax is `new ClassName { Property = value, ... }`. Object initializers can be combined with constructor arguments: `new ClassName(args) { Property = value }`. They are especially useful for `init`-only properties and `required` members.

**Beginner-Friendly Explanation**
An object initializer is **filling in the object’s properties at creation time** using `{ ... }`. It is concise and readable.

---

### Purposes

- To set properties at creation without separate assignments.
- To support `init` and `required` members.
- To improve readability of object creation.
- To avoid temporary variables.

---

### Syntax Rules and Structure

```
new ClassName { Property1 = value1, Property2 = value2 }
new ClassName(args) { Property = value }
```

**Component breakdown**

- `new ClassName` — The object creation.
- `{ ... }` — Initializer list.

#### Syntax Rules

- Properties/fields must be accessible and settable (or `init`-only).
- Constructor arguments can be combined with initializers.
- Initializers execute after the constructor body.

#### Constraints and Limitations

- Cannot set read-only fields (except via constructor).
- `required` members must be set in the initializer (or by a constructor annotated with `[SetsRequiredMembers]`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Object Initializer with `init` Properties

```csharp
using System;

public class Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}

class Program
{
    static void Main()
    {
        var p = new Person { Name = "Alice", Age = 30 };
        Console.WriteLine($"{p.Name}, {p.Age}");   // Alice, 30
    }
}
```

**Expected Output**

```
Alice, 30
```

**Why this output occurs**

- The initializer sets the `init`-only properties.

---

#### Example 2: Combined with Constructor

```csharp
using System;

public class Person
{
    public string Name { get; }
    public int Age { get; init; }

    public Person(string name) => Name = name;
}

class Program
{
    static void Main()
    {
        var p = new Person("Alice") { Age = 30 };
        Console.WriteLine($"{p.Name}, {p.Age}");   // Alice, 30
    }
}
```

**Expected Output**

```
Alice, 30
```

**Why this output occurs**

- The constructor sets `Name`; the initializer sets `Age`.

---

### Real-World Cases

- **Configuration** — Setting options at creation.
- **DTOs** — Initializing data properties.
- **Tests** — Concise test data construction.

---

### References

- Object and collection initializers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers

---

## 11. Constructor Best Practices

### Definitions

**Core Definition**
Constructor best practices are the design guidelines that ensure constructors are **safe, clear, and maintainable**, and that they establish valid object state without side effects.

**Technical Definition**
Best practices for constructors derive from the Framework Design Guidelines and the C# programming guide. Key recommendations include: **minimize the work done in constructors**; **use parameters to capture required dependencies and data**; **validate arguments in constructors** (throw `ArgumentNullException`, `ArgumentOutOfRangeException`); **prefer constructor injection for dependencies**; **do not call virtual methods from constructors**; **avoid side effects in constructors**; **use private constructors with static factories for controlled creation**; and **prefer primary constructors for simple initialization**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep constructors clean: **do not do too much**, **validate your inputs**, **do not call overridable methods**, and **use factories when creation needs to be controlled**.

---

### Purposes

- To ensure objects are created in a valid state.
- To keep constructors fast and predictable.
- To avoid subtle initialization-order bugs.
- To improve testability and maintainability.
- To align with .NET conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Validate arguments
public Person(string name)
{
    Name = name ?? throw new ArgumentNullException(nameof(name));
}

// Prefer constructor injection
public class Service(IRepository repo) { ... }

// Do not call virtual methods from constructors
// (base constructor runs before derived fields are initialized)

// Avoid side effects
// Use private constructors for controlled creation
// Prefer primary constructors for simple initialization
```

**Component breakdown**

- **Validation** — Check arguments and throw meaningful exceptions.
- **Injection** — Accept dependencies via constructor.
- **No virtual calls** — Avoid calling overridable members during construction.
- **No side effects** — Do not perform I/O or external actions in constructors.
- **Controlled creation** — Use private constructors and factories when needed.

#### Rules and Constraints

- **Do** validate constructor arguments.
- **Do** keep constructors focused on initialization.
- **Do** use constructor injection for dependencies.
- **Do not** call virtual methods from constructors.
- **Do not** perform I/O or long-running work in constructors.
- **Do not** expose constructors that create partially initialized objects.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Validated Constructor

```csharp
using System;

public class Person
{
    public string Name { get; }

    public Person(string name)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
    }
}

class Program
{
    static void Main()
    {
        try { new Person(null!); }
        catch (ArgumentNullException ex) { Console.WriteLine(ex.ParamName); } // name
    }
}
```

**Expected Output**

```
name
```

**Why this output occurs**

- The constructor validates `name` and throws a meaningful exception.

---

#### Example 2: Constructor Injection

```csharp
using System;

public interface ILogger { void Log(string message); }
public class ConsoleLogger : ILogger { public void Log(string m) => Console.WriteLine(m); }

public class Service
{
    private readonly ILogger _logger;
    public Service(ILogger logger) => _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    public void Run() => _logger.Log("Running");
}

class Program
{
    static void Main()
    {
        var s = new Service(new ConsoleLogger());
        s.Run();   // Running
    }
}
```

**Expected Output**

```
Running
```

**Why this output occurs**

- The constructor receives the dependency and stores it in a readonly field.

---

### Real-World Cases

- **Dependency injection** — Constructors receive services.
- **Domain models** — Constructors validate and enforce invariants.
- **Factory patterns** — Private constructors with public static factories.

---

### References

- Constructor design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/constructor
- Constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors
- Instance constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors
- Static constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors
- Private constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/private-constructors

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Primary constructors | Introduced in **C# 12**; version-specific. |
| Parameterless struct constructors | Introduced in **C# 10**; earlier versions disallow explicit parameterless struct constructors. |
| `required` members | Introduced in **C# 11**; interacts with constructors via `[SetsRequiredMembers]`. |
| `init` accessors | Introduced in **C# 9**; used with object initializers. |
| Static constructors | Available since C# 1.0; exceptions render the type unusable. |
| Private constructors | Available since C# 1.0. |
| Copy constructors | **Not auto-generated** for classes/structs; records get compiler-generated copy support. |
| Calling virtual methods from constructors | **Discouraged**; derived overrides run before derived fields are initialized. |

---

## References (Consolidated)

- Constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors
- Instance constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors
- Static constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors
- Private constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/private-constructors
- Primary constructors — C# 12 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12#primary-constructors
- Primary constructors — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/instance-constructors#primary-constructors
- Primary constructors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-12.0/primary-constructors
- Object and collection initializers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers
- with expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression
- Object.MemberwiseClone — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.memberwiseclone
- this keyword — C# reference (constructor chaining) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/this
- base keyword — C# reference (constructor calls) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base
- Constructors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1511-constructors
- Constructor design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/constructor

---