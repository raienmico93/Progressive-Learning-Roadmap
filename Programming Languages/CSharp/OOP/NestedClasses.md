# C# Nested Classes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **nested class** (also called a **nested type**) in C# is a class defined **within another class, struct, or interface**. The enclosing type acts as a **scope and access boundary**, allowing the nested class to be hidden from external code or granted controlled visibility. Unlike Java’s “inner classes,” a C# nested class does **not** carry an implicit reference to an instance of its enclosing type—it behaves like a top-level class that happens to be declared inside another type for organisational and access-control reasons.

**Technical Definition**
A nested type is a type that you define within a class, struct, or interface. Regardless of whether the outer type is a class, interface, or struct, nested types **default to `private`**; they are accessible only from their containing type. Nested types of a **class** can be declared `public`, `protected`, `internal`, `protected internal`, `private`, or `private protected`. Nested types of a **struct** can be `public`, `internal`, or `private`. A nested type has access to **all members accessible to its containing type**, including `private` and `protected` members of the containing type and inherited `protected` members of base types. The nested type’s full name is qualified by the enclosing type (e.g., `Container.Nested`). Crucially, a nested type and its containing type **do not have a special relationship with regard to `this` access**: `this` inside a nested class refers to the nested instance, **not** to an instance of the containing type. To access the enclosing instance, the nested type must receive it explicitly, typically through a constructor parameter.

**Beginner-Friendly Explanation**
A nested class is a **class inside another class**. Think of it as a **room inside a house**: the room has its own furniture (its own fields and methods), and it can look into the rest of the house (it can access the house’s private members), but from outside the house, you cannot see the room unless the house explicitly opens a door to it. By default, the room is **hidden**—only the house knows about it. A nested class is **not** a subclass and has **no automatic link** to an instance of the outer class; it is simply a class that lives in the outer class’s scope. This makes nested classes perfect for **small helper types** that are only meaningful in the context of their container—like a `Node` class inside a `LinkedList`, or a `Builder` inside a `Configuration` class.

---

### Key Characteristics

- **Declared inside another type** — Nested classes can be declared inside a class, struct, or interface.
- **Default `private` accessibility** — Unlike top-level types (which default to `internal`), nested types default to `private`.
- **Full range of access modifiers (for class containers)** — Nested types of a class can be `public`, `protected`, `internal`, `protected internal`, `private`, or `private protected`.
- **No implicit outer instance** — A C# nested class does **not** hold a reference to an instance of its enclosing type (unlike Java inner classes).
- **Access to enclosing private members** — A nested type can access all members accessible to its containing type, including `private` and `protected` members.
- **Qualified naming** — The full name of a nested type is `Outer.Nested`; external code must use this qualification.
- **`this` refers to the nested instance** — `this` inside a nested class refers to the nested object, not the outer object; the outer instance must be passed explicitly if needed.
- **No special inheritance relationship** — A nested class is not a subclass of its containing type; it is simply scoped within it.
- **Compiler-generated nested types** — The C# compiler itself uses nested types for iterators and anonymous methods.
- **CA1034 warning** — Making a nested type externally visible violates code quality rule CA1034 (“Nested types should not be visible”), signalling that the design should be reconsidered.

---

### Prerequisites

- Understanding of C# classes, structs, and interfaces.
- Familiarity with access modifiers (`public`, `private`, `protected`, `internal`, `protected internal`, `private protected`).
- Knowledge of instance members, `this`, and object construction.
- Awareness of namespaces and type qualification.
- Basic understanding of helper classes and encapsulation.

---

### Related Programming Areas

- **Encapsulation** — Nested classes provide tighter access control than namespace-level types.
- **Helper types** — Small, implementation-specific types (e.g., `Node`, `Builder`, `Enumerator`) are often nested.
- **Compiler-generated code** — Iterators and anonymous methods produce nested types.
- **Partial classes** — Nested classes can be split across files using `partial`.
- **Design guidelines** — CA1034 discourages exposing nested types publicly; understanding when to nest is a design decision.
- **Java interop** — C# nested classes are **not** Java inner classes; the difference matters for developers coming from Java.

---

### Core Concepts / Features

1. Nested Classes (Overarching Concept)
2. Inner Classes (and the Java Distinction)
3. Nested Class Access (Modifiers and Rules)
4. Nested Class Scope (Qualification and `this`)
5. Nested Class Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Nested Classes (Overarching Concept)

### Definitions

**Core Definition**
A nested class is a class declared inside another class, struct, or interface, creating a scoped type that is hidden or exposed according to its access modifier.

**Technical Definition**
A nested type is a type that you define within a class, struct, or interface. The nested type’s accessibility is controlled by an access modifier; if none is specified, it defaults to `private`. Nested types have access to all members accessible to their containing type. They are qualified by the enclosing type’s name (`Outer.Nested`). All types—classes, structs, interfaces, delegates, and enums—can be nested within either a class or a struct.

**Beginner-Friendly Explanation**
A nested class is a **class defined inside another class**. It is a way to keep a helper type close to where it is used and to hide it from the rest of the program.

---

### Purposes

- To **hide implementation details** from external code by using `private` or `internal` accessibility.
- To **group a helper type** with the type that uses it, improving locality and readability.
- To **access the enclosing type’s private members**, enabling tight cooperation between the two types.
- To **avoid namespace pollution** with types that are only meaningful in one context.
- To **express a strong logical relationship** between the nested type and its container (e.g., `LinkedList.Node`).

---

### Syntax Rules and Structure

```
[access modifier] class OuterClass
{
    [access modifier] class NestedClass
    {
        // nested class members
    }
}
```

**Component breakdown**

- `OuterClass` — The containing type.
- `NestedClass` — The nested type declared inside the outer type.
- `access modifier` — Optional; defaults to `private` for the nested type.

#### Syntax Rules

- Nested types default to `private`.
- Nested types of a class can use any access modifier applicable to a class member.
- Nested types of a struct can be `public`, `internal`, or `private`.
- The nested type’s full name is `Outer.Nested`.
- Nested types can be `static`, `abstract`, `sealed`, `partial`, or `unsafe`.

#### Constraints and Limitations

- A nested type cannot access instance members of the enclosing type through `this` alone; the enclosing instance must be passed explicitly.
- Externally visible nested types trigger CA1034.
- A `protected` nested type inside a sealed class triggers CS0628.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Private Nested Helper Class

```csharp
using System;

public class LinkedList
{
    // Nested class, private by default
    private class Node
    {
        public int Value;
        public Node? Next;

        public Node(int value) => Value = value;
    }

    private Node? _head;

    public void Add(int value)
    {
        var newNode = new Node(value);   // accessible inside LinkedList
        if (_head == null) { _head = newNode; return; }
        var current = _head;
        while (current.Next != null) current = current.Next;
        current.Next = newNode;
    }

    public void Print()
    {
        var current = _head;
        while (current != null)
        {
            Console.Write(current.Value + " ");
            current = current.Next;
        }
        Console.WriteLine();
    }
}

class Program
{
    static void Main()
    {
        var list = new LinkedList();
        list.Add(10); list.Add(20); list.Add(30);
        list.Print();   // 10 20 30

        // Node is not accessible here:
        // LinkedList.Node node = null; // Compile error: inaccessible
    }
}
```

**Expected Output**

```
10 20 30 
```

**Why this output occurs**

- `Node` is a **private nested class**; only `LinkedList` can use it. External code cannot see or instantiate `Node`, enforcing encapsulation.

---

#### Example 2: Public Nested Type (with CA1034 caveat)

```csharp
using System;

public class Container
{
    public class Nested
    {
        public void Hello() => Console.WriteLine("Hello from Nested");
    }
}

class Program
{
    static void Main()
    {
        var n = new Container.Nested();   // qualified name
        n.Hello();   // Hello from Nested
    }
}
```

**Expected Output**

```
Hello from Nested
```

**Why this output occurs**

- `Nested` is **public**, so external code can access it using the qualified name `Container.Nested`. Note that this design triggers CA1034 (“Nested types should not be visible”) and should be justified.

---

### Real-World Cases

- **`LinkedList<T>.Node`** — The node type is an implementation detail of the linked list.
- **`Dictionary<TKey,TValue>.Enumerator`** — The enumerator is a nested struct.
- **Configuration builders** — A `Builder` nested inside a `Configuration` class.
- **Compiler-generated iterators** — Iterator methods generate nested state-machine types.

---

### References

- Nested types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/nested-types
- Classes — C# language specification (nested types) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- CA1034: Nested types should not be visible — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1034

---

## 2. Inner Classes (and the Java Distinction)

### Definitions

**Core Definition**
“Inner class” is a term from **Java** describing a non-static nested class that carries an **implicit reference** to an instance of its enclosing class. C# has **no equivalent**; C# nested classes are always like Java’s **`static` nested classes**—they do not hold an implicit outer instance.

**Technical Definition**
C# does not have Java-style inner classes. A C# nested class is a **nested type** with no implicit reference to an enclosing instance. In Java, a non-static inner class holds a hidden reference to the outer instance, allowing it to access the outer instance’s members directly. In C#, `this` inside a nested class refers only to the nested instance; to access the enclosing instance’s members, the nested type must receive the enclosing instance explicitly (typically via a constructor parameter). The access rules also differ: in Java, the outer class can access the nested class’s private members and vice versa; in C#, the nested class can access the outer class’s private members, but **not** the other way around. C# uses **anonymous methods and lambda expressions** for the scenarios where Java developers would use anonymous inner classes.

**Beginner-Friendly Explanation**
In Java, an “inner class” is a class that is **automatically tied to an outer object**—it can reach out and touch the outer object’s data without being told which outer object it belongs to. In C#, nested classes are **independent**; they do not automatically know about any outer object. If a nested class needs to talk to an outer object, you must hand it that object explicitly. C# nested classes are therefore **simpler and more predictable**, but they require a little more ceremony when outer-instance access is needed.

---

### Purposes

- To clarify that C# nested classes are **not** Java inner classes.
- To guide developers coming from Java in translating their designs.
- To explain why C# nested classes do not need an implicit outer reference.
- To highlight the **explicit outer-instance passing** pattern in C#.

---

### Syntax Rules and Structure (Java vs. C#)

| Aspect | Java inner class (non-static) | C# nested class |
|---|---|---|
| **Outer instance reference** | Implicit | **None** |
| **`this` inside nested** | Refers to nested; outer accessed via `Outer.this` | Refers to nested only |
| **Outer access to nested private members** | Yes | **No** (nested private members are private to the nested type) |
| **Nested access to outer private members** | Yes | **Yes** |
| **Static nested class** | No implicit outer instance | All C# nested classes behave like this |

**Component breakdown**

- **Implicit outer reference** — Only in Java.
- **Explicit passing** — Required in C# when the nested type needs the outer instance.

#### Syntax Rules

- In C#, to access the enclosing instance from a nested type, pass `this` (or the outer instance) to the nested type’s constructor.
- The nested type stores the outer reference in a field.

#### Constraints and Limitations

- C# nested classes cannot use `Outer.this` (that syntax does not exist).
- Anonymous inner classes in Java are replaced by lambdas and anonymous methods in C#.

---

### Annotated Code Examples and Expected Outputs

#### Example: Passing the Outer Instance Explicitly

```csharp
using System;

public class Outer
{
    private int _value = 123;

    public class Nested
    {
        private readonly Outer _outer;

        public Nested(Outer outer) => _outer = outer;

        public void ShowOuterValue() => Console.WriteLine(_outer._value);
    }

    public void Run()
    {
        var nested = new Nested(this);   // pass outer instance
        nested.ShowOuterValue();          // 123
    }
}

class Program
{
    static void Main()
    {
        new Outer().Run();   // 123
    }
}
```

**Expected Output**

```
123
```

**Why this output occurs**

- The nested class receives the outer instance through its constructor and accesses the private `_value` field through that reference.

---

### Real-World Cases

- **Porting Java code to C#** — Java inner classes must be refactored to pass the outer instance explicitly.
- **Event handling** — Java anonymous inner classes become C# lambdas.
- **Builder patterns** — C# builders are often nested `static` classes or separate classes, not inner classes.

---

### References

- What are the fundamental differences between Java and C# in terms of inner/local/anonymous classes? — Stack Overflow (Jon Skeet’s answer) - https://stackoverflow.com/questions/686114/what-are-the-fundamental-differences-between-java-and-c-in-terms-of-inner-local-a
- C# nested classes are like C++ nested classes, not Java inner classes — Stack Overflow - https://stackoverflow.com/questions/686114/what-are-the-fundamental-differences-between-java-and-c-in-terms-of-inner-local-a
- Programming C# — O’Reilly (nested classes are roughly equivalent to Java’s static inner classes) - https://www.oreilly.com/library/view/programming-c/0596001177/ch04s03.html

---

## 3. Nested Class Access (Modifiers and Rules)

### Definitions

**Core Definition**
Nested class access refers to the **access modifiers** that can be applied to a nested type and the **access rules** that govern which code can see and use the nested type.

**Technical Definition**
Regardless of whether the outer type is a class, interface, or struct, nested types **default to `private`**. Nested types of a **class** can be declared `public`, `protected`, `internal`, `protected internal`, `private`, or `private protected`. Nested types of a **struct** can be `public`, `internal`, or `private`. A nested type has access to all members accessible to its containing type, including `private` and `protected` members. Making a nested type externally visible violates **CA1034**. Defining a `protected`, `protected internal`, or `private protected` nested class inside a **sealed class** generates compiler warning **CS0628** (“new protected member declared in sealed class”).

**Beginner-Friendly Explanation**
The access modifier on a nested class controls **who can see it**. By default, it is **private**—only the outer class can see it. You can make it **public** (anyone), **internal** (same assembly), **protected** (derived classes), and so on. The nested class can **look inside** the outer class (even at private members), but the outer class **cannot** look inside the nested class’s private members.

---

### Purposes

- To control the visibility of helper types.
- To enable or restrict access from derived classes and external assemblies.
- To enforce encapsulation of implementation details.
- To satisfy code analysis rules (CA1034, CS0628).
- To grant the nested type access to the enclosing type’s private state.

---

### Syntax Rules and Structure (Access Modifier Matrix)

| Modifier | Class container | Struct container | Notes |
|---|---|---|---|
| `private` | ✔️ (default) | ✔️ (default) | Accessible only within the containing type |
| `public` | ✔️ | ✔️ | Accessible everywhere (triggers CA1034) |
| `internal` | ✔️ | ✔️ | Accessible within the assembly |
| `protected` | ✔️ | ❌ | Accessible in derived classes (CS0628 if sealed) |
| `protected internal` | ✔️ | ❌ | Union of protected and internal |
| `private protected` | ✔️ | ❌ | Intersection of protected and internal |

**Component breakdown**

- **Default `private`** — Nested types are hidden by default.
- **`public`** — Exposes the nested type; CA1034 warning.
- **`internal`** — Visible within the assembly.
- **`protected` family** — Visible to derived classes (class containers only).

#### Syntax Rules

- Nested types default to `private`.
- Class containers allow the full range of access modifiers.
- Struct containers allow only `public`, `internal`, and `private`.
- `protected` nested types in sealed classes trigger CS0628.

#### Constraints and Limitations

- CA1034 warns against externally visible nested types.
- The outer type cannot access the nested type’s private members (unlike Java).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Private vs. Internal vs. Public Nested Types

```csharp
using System;

public class Container
{
    private class PrivateNested { public void Hello() => Console.WriteLine("Private"); }
    internal class InternalNested { public void Hello() => Console.WriteLine("Internal"); }
    public class PublicNested { public void Hello() => Console.WriteLine("Public"); }

    public void UsePrivate()
    {
        var p = new PrivateNested();   // OK: inside Container
        p.Hello();                      // Private
    }
}

class Program
{
    static void Main()
    {
        var c = new Container();
        c.UsePrivate();                      // Private

        var i = new Container.InternalNested(); // OK: same assembly
        i.Hello();                            // Internal

        var p = new Container.PublicNested();    // OK: public
        p.Hello();                               // Public

        // var priv = new Container.PrivateNested(); // Compile error
    }
}
```

**Expected Output**

```
Private
Internal
Public
```

**Why this output occurs**

- `PrivateNested` is accessible only inside `Container`; `InternalNested` is accessible within the assembly; `PublicNested` is accessible everywhere.

---

#### Example 2: Protected Nested Class

```csharp
using System;

public class Base
{
    protected class ProtectedNested
    {
        public void Hello() => Console.WriteLine("Protected");
    }
}

public class Derived : Base
{
    public void UseProtected()
    {
        var p = new ProtectedNested();   // OK: derived class
        p.Hello();                        // Protected
    }
}

class Program
{
    static void Main()
    {
        new Derived().UseProtected();   // Protected
    }
}
```

**Expected Output**

```
Protected
```

**Why this output occurs**

- `ProtectedNested` is accessible within derived classes, so `Derived` can use it.

---

### Real-World Cases

- **Private helper types** — `private class Node` inside a collection.
- **Internal implementation types** — `internal class Builder` inside a public class.
- **Protected extension points** — `protected class Options` for derived classes.

---

### References

- Nested types — C# Programming Guide (access modifiers) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/nested-types
- CA1034: Nested types should not be visible — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1034
- CS0628: New protected member declared in sealed class — C# compiler warning - https://learn.microsoft.com/en-us/dotnet/csharp/misc/cs0628

---

## 4. Nested Class Scope (Qualification and `this`)

### Definitions

**Core Definition**
Nested class scope defines **how the nested type is named and accessed** (qualification) and **how `this` behaves** inside the nested class (it refers to the nested instance, not the outer instance).

**Technical Definition**
The full name of a nested type is `Outer.Nested`. External code must use this qualified name to create instances or refer to the nested type. Inside the nested type, `this` refers to the nested instance, and there is **no implicit reference** to an instance of the containing type. To access the enclosing instance, the nested type must receive it explicitly, typically through a constructor parameter. A nested type can access **static** members of the enclosing type directly. The nested type also has access to all members accessible to its containing type, including `private` and `protected` members.

**Beginner-Friendly Explanation**
- **Qualification** — From outside, you must say `Outer.Nested` to reach the nested class.
- **`this`** — Inside the nested class, `this` is the nested object, not the outer object. If you need the outer object, you must pass it in.

---

### Purposes

- To understand how to reference nested types from outside.
- To correctly access the enclosing instance when needed.
- To avoid confusion about what `this` means inside a nested class.
- To use nested types as part of a public API (qualified names).

---

### Syntax Rules and Structure (Scope Rules)

| Aspect | Rule |
|---|---|
| **Qualified name** | `Outer.Nested` |
| **`this` inside nested** | Refers to the nested instance only |
| **Implicit outer instance** | None |
| **Access to outer instance members** | Requires explicit outer reference |
| **Access to outer static members** | Direct (no instance needed) |
| **Access to outer private members** | Allowed (through explicit outer reference for instance members; directly for static) |

**Component breakdown**

- **Qualified name** — Required for external access.
- **`this`** — Nested instance only.
- **Explicit outer reference** — Constructor parameter pattern.

#### Syntax Rules

- External code uses `Outer.Nested` to refer to the nested type.
- Inside the nested type, `this` does not reach the outer instance.
- Static members of the outer type are accessible directly.

#### Constraints and Limitations

- There is no `Outer.this` syntax in C# (unlike Java).
- The outer instance must be passed explicitly.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Qualified Name and `this` Behaviour

```csharp
using System;

public class Outer
{
    private int _outerValue = 10;

    public class Nested
    {
        private int _nestedValue = 20;

        public void Show()
        {
            Console.WriteLine($"Nested this: {this._nestedValue}");   // 20
            // Console.WriteLine(this._outerValue); // Compile error
        }
    }
}

class Program
{
    static void Main()
    {
        var nested = new Outer.Nested();   // qualified name
        nested.Show();                      // Nested this: 20
    }
}
```

**Expected Output**

```
Nested this: 20
```

**Why this output occurs**

- `this` inside `Nested` refers to the `Nested` instance; the outer instance is not accessible through `this`.

---

#### Example 2: Passing the Outer Instance

```csharp
using System;

public class Outer
{
    private int _value = 42;

    public class Nested
    {
        private readonly Outer _outer;

        public Nested(Outer outer) => _outer = outer;

        public void Show() => Console.WriteLine(_outer._value);
    }

    public void Run()
    {
        var nested = new Nested(this);   // pass outer instance
        nested.Show();                    // 42
    }
}

class Program
{
    static void Main()
    {
        new Outer().Run();   // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- The nested type receives the outer instance explicitly and accesses the private field through it.

---

### Real-World Cases

- **Public nested types** — Using `Outer.Nested` in API signatures.
- **Helper types needing outer state** — Passing the outer instance to a nested helper.
- **Static nested helpers** — Accessing outer static members directly without an instance.

---

### References

- Nested types — C# Programming Guide (qualified names) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/nested-types
- Classes — C# language specification (`this` access in nested types) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes

---

## 5. Nested Class Best Practices

### Definitions

**Core Definition**
Nested class best practices are the design guidelines that ensure nested types are used **appropriately, sparingly, and maintainably**, without abusing access control or cluttering APIs.

**Technical Definition**
Best practices for nested classes derive from the Framework Design Guidelines, code analysis rules (CA1034), and community guidance. Key recommendations include: **use nested types for helper classes that are only used by the containing type**; **avoid making nested types publicly visible** (CA1034); **consider a separate top-level type or a nested namespace instead of nesting solely to avoid namespace pollution**; **use `private` or `internal` accessibility for implementation details**; **be aware that a nested type should exist because of its stronger access control restrictions or because it must access private members of the containing type**; **split large nested types into separate files using partial classes**; and **prefer static nested types when no outer instance is needed**.

**Beginner-Friendly Explanation**
Best practices for nested classes are simple: **use them for small helpers that belong to one class**, **do not expose them publicly**, and **do not nest just to avoid creating a new namespace**. If the nested class is big, consider moving it out.

---

### Purposes

- To prevent API clutter and CA1034 violations.
- To keep helper types hidden and focused.
- To avoid deep nesting and large files.
- To use nested types for the right reasons (access control, private-member access).
- To align with .NET design conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Use nested types for private helpers
public class LinkedList
{
    private class Node { ... }   // hidden implementation detail
}

// Avoid publicly visible nested types (CA1034)
// Instead, use a top-level type or a nested namespace

// Use partial classes to split large nested types into separate files
public partial class Outer
{
    private class BigHelper { ... }
}

// Prefer static nested types when no outer instance is needed
public class Outer
{
    public static class Helpers { ... }
}
```

**Component breakdown**

- **Private helpers** — The primary legitimate use of nested types.
- **Avoid public nesting** — CA1034 recommends against externally visible nested types.
- **Partial classes** — Split large nested types across files for maintainability.
- **Static nested types** — When the nested type does not need an outer instance.

#### Rules and Constraints

- **Do** use nested types for private helper classes.
- **Do** use `private` or `internal` accessibility for implementation details.
- **Do** split large nested types with `partial` classes.
- **Do not** make nested types publicly visible without a compelling reason.
- **Do not** nest types solely to avoid namespace pollution; use a nested namespace or a separate file instead.
- **Do not** create deeply nested hierarchies of types.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Nested Helper

```csharp
using System;

public class Configuration
{
    // Private nested helper, hidden from external code
    private class Validator
    {
        public static bool IsValid(string value) =>
            !string.IsNullOrWhiteSpace(value) && value.Length <= 100;
    }

    public string Name { get; }

    public Configuration(string name)
    {
        if (!Validator.IsValid(name))
            throw new ArgumentException("Invalid name", nameof(name));
        Name = name;
    }
}

class Program
{
    static void Main()
    {
        var config = new Configuration("MyApp");
        Console.WriteLine(config.Name);   // MyApp
    }
}
```

**Expected Output**

```
MyApp
```

**Why this output occurs**

- `Validator` is a **private nested helper** used only by `Configuration`; it is hidden from external code and keeps validation logic close to where it is used.

---

### Real-World Cases

- **`LinkedList<T>.Node`** — A classic private nested helper.
- **`Dictionary<TKey,TValue>.Enumerator`** — A nested struct for enumeration.
- **Configuration builders** — Nested builder classes that are `internal` or `private`.
- **Compiler-generated state machines** — Nested types for iterators and async methods.

---

### References

- Nested types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/nested-types
- CA1034: Nested types should not be visible — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1034
- Using partial classes with nested classes for maintainability — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/marcelolr/using-partial-classes-with-nested-classes-for-maintainability
- C# 9.0 in a Nutshell — Nested Types (guidelines) - https://archive.org/stream/csharp-9-in-a-nutshell-the-definitive-reference

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Nested types default to `private` | **Language rule**; stable across all C# versions. |
| `protected` nested types in sealed classes | **CS0628 warning**; avoid. |
| Externally visible nested types | **CA1034 warning**; design smell. |
| Java-style inner classes | **Not supported** in C#; nested classes have no implicit outer reference. |
| `Outer.this` syntax | **Does not exist** in C#. |
| Compiler-generated nested types | Used for iterators and anonymous methods; **implementation detail**. |
| Partial nested classes | Allowed; useful for splitting large nested types across files. |

---

## References (Consolidated)

- Nested types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/nested-types
- Classes — C# language specification (nested types) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- CA1034: Nested types should not be visible — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1034
- CS0628: New protected member declared in sealed class — C# compiler warning - https://learn.microsoft.com/en-us/dotnet/csharp/misc/cs0628
- What are the fundamental differences between Java and C# in terms of inner/local/anonymous classes? — Stack Overflow (Jon Skeet) - https://stackoverflow.com/questions/686114/what-are-the-fundamental-differences-between-java-and-c-in-terms-of-inner-local-a
- C# nested classes are like C++ nested classes, not Java inner classes — Stack Overflow - https://stackoverflow.com/questions/686114/what-are-the-fundamental-differences-between-java-and-c-in-terms-of-inner-local-a
- Programming C# — O’Reilly (nested classes ≈ Java static inner classes) - https://www.oreilly.com/library/view/programming-c/0596001177/ch04s03.html
- Using partial classes with nested classes for maintainability — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/marcelolr/using-partial-classes-with-nested-classes-for-maintainability
- C# 9.0 in a Nutshell — Nested Types (guidelines) - https://archive.org/stream/csharp-9-in-a-nutshell-the-definitive-reference

---