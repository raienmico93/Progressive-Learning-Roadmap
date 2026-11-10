# C# Generic Constraints — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Generic constraints are **rules** applied to type parameters using the `where` clause. They restrict the set of types that can be used as type arguments, and they **declare the capabilities** that a type argument must have. Without constraints, the compiler knows nothing about `T` beyond the members of `System.Object`. Constraints are the mechanism that unlocks the full power of generics: they enable generic code to access methods, properties, operators, and constructors of `T`, while guaranteeing type safety at compile time.

**Technical Definition**
Constraints declare the capabilities that a type argument must have. The `where` clause is used to specify constraints on type parameters in generic types, methods, delegates, and local functions. A type parameter can have **at most one primary constraint** (`class`, `class?`, `struct`, `unmanaged`, `notnull`, or `default`), followed by **zero or more base class and interface constraints**, and finally the **`new()` constructor constraint** (which must appear last). Some constraints are **mutually exclusive** and cannot be combined. All constraints for a single type parameter must appear in a **single `where` clause**. Constraints are enforced at compile time: using a type argument that does not satisfy the constraints produces a compiler error. Since **C# 11 / .NET 7**, interfaces can declare `static abstract` members, enabling **generic math** constraints such as `where T : INumber<T>`, which allow generic algorithms to use arithmetic operators.

**Beginner-Friendly Explanation**
Generic constraints are **rules about what `T` can be**. Without constraints, `T` is a mystery type—the compiler only knows it is an `object`. Constraints let you say “`T` must be comparable,” “`T` must have a parameterless constructor,” or “`T` must be a value type.” Once you state the rule, the compiler lets you use the capabilities that the rule guarantees. Think of constraints as a **job requirement**: “The type filling this role must be able to do X, Y, and Z.” Constraints make generic code both **safer** (invalid type arguments are rejected at compile time) and **more powerful** (you can call methods on `T` that the constraints guarantee exist).

---

### Key Characteristics

- **`where` clause syntax** — Constraints are specified with the `where` keyword followed by the type parameter and a colon.
- **Order matters** — Primary constraints come first, then base class, then interfaces, then `new()`.
- **Mutual exclusivity** — `class`, `struct`, `unmanaged`, `notnull`, and `default` are mutually exclusive; you can use at most one.
- **`struct` implies `new()`** — The `struct` constraint already requires a parameterless constructor, so `new()` cannot be combined with `struct` or `unmanaged`.
- **Base class before interfaces** — The base class constraint must appear before interface constraints.
- **`new()` must be last** — The `new()` constraint must be the final constraint in the list.
- **Single `where` clause per type parameter** — Multiple `where` clauses for the same type parameter are invalid; combine all constraints into one clause.
- **Generic math (C# 11+)** — `where T : INumber<T>` enables arithmetic operators on generic types.
- **Compile-time enforcement** — Type arguments that violate constraints are rejected by the compiler.

---

### Prerequisites

- Basic understanding of C# generics (type parameters, generic classes, methods).
- Familiarity with interfaces, base classes, and inheritance.
- Knowledge of value types vs. reference types and nullable types.
- Awareness of the `where` clause in generic definitions.
- Basic understanding of `System.Object` as the root of the type hierarchy.

---

### Related Programming Areas

- **Generic algorithms** — Constraints enable algorithms that rely on comparison, arithmetic, or construction.
- **Generic math** — C# 11+ generic math interfaces (`INumber<T>`, `IAdditionOperators<T,T,T>`) use static abstract constraints.
- **Dependency injection** — Generic constraints ensure that resolved types have the required constructors or interfaces.
- **Repository patterns** — `where T : class, IEntity` enforces that entities are reference types implementing an interface.
- **Factory methods** — `where T : new()` enables generic factories.

---

### Core Concepts / Features

1. Reference Type Constraint (`class`)
2. Value Type Constraint (`struct`)
3. Not-Null Constraint (`notnull`)
4. Unmanaged Constraint (`unmanaged`)
5. Constructor Constraint (`new()`)
6. Base Class Constraint
7. Interface Constraint
8. Type Parameter Constraint (`where T : U`)
9. Multiple Constraints
10. Constraint Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Reference Type Constraint (`class`)

### Definitions

**Core Definition**
The `class` constraint requires that the type argument be a **reference type**. In a nullable context, it requires a **non-nullable** reference type; `class?` allows nullable or non-nullable reference types.

**Technical Definition**
`where T : class` specifies that the type argument must be a reference type. This constraint also applies to any class, interface, delegate, or array type. In a nullable context, `T` must be a non-nullable reference type. `where T : class?` specifies that the type argument must be a nullable or non-nullable reference type, including classes, interfaces, delegates, and arrays (including records). The `class` constraint is a **primary constraint** and must appear first in the constraint list. It is **mutually exclusive** with `struct`, `unmanaged`, `notnull`, and `default`.

**Beginner-Friendly Explanation**
`class` means **“T must be a reference type”** —something you can set to `null`. `class?` means “T can be a reference type that might be `null`.”

---

### Purposes

- To ensure that `T` is a reference type, enabling null-related operations and reference semantics.
- To distinguish reference-type generics from value-type generics.
- To enable APIs that accept classes, interfaces, delegates, or arrays.
- To provide compile-time documentation of the type’s reference-type nature.

---

### Syntax Rules and Structure

```
where T : class
where T : class?
```

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The type parameter.
- `:` — Separator.
- `class` — Requires a non-nullable reference type.
- `class?` — Allows nullable or non-nullable reference types.

#### Syntax Rules

- `class` is a primary constraint; it must appear first.
- `class` and `class?` are mutually exclusive with `struct`, `unmanaged`, `notnull`, and `default`.
- `class?` allows nullable reference types (C# 8+ nullable context).

#### Constraints and Limitations

- Cannot be combined with `struct` or `unmanaged`.
- In a non-nullable context, `class` requires a non-nullable reference type.

---

### Annotated Code Examples and Expected Outputs

#### Example: `class` Constraint

```csharp
using System;

public class ReferenceContainer<T> where T : class
{
    public T? Item { get; set; }
}

class Program
{
    static void Main()
    {
        var stringContainer = new ReferenceContainer<string> { Item = "Hello" };
        Console.WriteLine(stringContainer.Item);   // Hello

        // var intContainer = new ReferenceContainer<int>(); // Compile error: int is not a reference type
    }
}
```

**Expected Output**

```
Hello
```

**Why this output occurs**

- `string` satisfies the `class` constraint; `int` does not, so the second instantiation is rejected at compile time.

---

### Real-World Cases

- **Reference-type repositories** — `where T : class` ensures entities are reference types.
- **Caching** — Reference-type caches that can hold `null`.
- **Null-safe containers** — Generic containers that explicitly handle reference types.

---

### References

- Constraints on type parameters — `class` constraint - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#reference-type-constraint

---

## 2. Value Type Constraint (`struct`)

### Definitions

**Core Definition**
The `struct` constraint requires that the type argument be a **non-nullable value type**. It **implies** the `new()` constraint and cannot be combined with it.

**Technical Definition**
`where T : struct` specifies that the type argument must be a non-nullable value type, which includes `record struct` types. Because all value types have an accessible parameterless constructor (declared or implicit), the `struct` constraint **implies the `new()` constraint** and **cannot be combined with `new()`**. The `struct` constraint also **cannot be combined with `unmanaged`**. It is a **primary constraint** and must appear first.

**Beginner-Friendly Explanation**
`struct` means **“T must be a value type”** —something like `int`, `double`, or a custom struct. It cannot be `null`.

---

### Purposes

- To ensure that `T` is a value type, enabling stack-friendly and copy-semantics operations.
- To avoid heap allocations in generic code.
- To distinguish value-type generics from reference-type generics.
- To enable APIs that require value-type semantics.

---

### Syntax Rules and Structure

```
where T : struct
```

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The type parameter.
- `:` — Separator.
- `struct` — Requires a non-nullable value type.

#### Syntax Rules

- `struct` is a primary constraint; it must appear first.
- `struct` implies `new()`; it cannot be combined with `new()`.
- `struct` cannot be combined with `unmanaged`.
- `struct` is mutually exclusive with `class`, `class?`, `notnull`, and `default`.

#### Constraints and Limitations

- Cannot be combined with `new()` (redundant).
- Cannot be combined with `unmanaged` (mutually exclusive).
- Nullable value types (`int?`) do not satisfy `struct` (they are `Nullable<T>`, which is a struct but the type argument itself is a nullable value type—actually `int?` is a value type, but the constraint checks the type argument; `Nullable<int>` is a struct and does satisfy `struct`, but `int?` is `Nullable<int>`).

---

### Annotated Code Examples and Expected Outputs

#### Example: `struct` Constraint

```csharp
using System;

public class ValueContainer<T> where T : struct
{
    public T Item { get; set; }
}

class Program
{
    static void Main()
    {
        var intContainer = new ValueContainer<int> { Item = 42 };
        Console.WriteLine(intContainer.Item);   // 42

        // var stringContainer = new ValueContainer<string>(); // Compile error: string is not a value type
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- `int` satisfies the `struct` constraint; `string` does not.

---

### Real-World Cases

- **Numeric algorithms** — Generic math over value types.
- **Performance-sensitive containers** — Avoiding heap allocation.
- **Struct-based caches** — Storing small value-type data.

---

### References

- Constraints on type parameters — `struct` constraint - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#value-type-constraint

---

## 3. Not-Null Constraint (`notnull`)

### Definitions

**Core Definition**
The `notnull` constraint requires that the type argument be a **non-nullable type**—either a non-nullable reference type or a non-nullable value type.

**Technical Definition**
`where T : notnull` specifies that the type argument must be a non-nullable type. The type argument can be a non-nullable reference type or a non-nullable value type. This constraint is primarily useful in **nullable contexts** (C# 8+) to ensure that `T` is not a nullable type. `notnull` is a **primary constraint** and is **mutually exclusive** with `class`, `class?`, `struct`, `unmanaged`, and `default`.

**Beginner-Friendly Explanation**
`notnull` means **“T cannot be null”** —it can be a non-nullable reference type or a non-nullable value type.

---

### Purposes

- To ensure that `T` is non-nullable, enabling null-safe generic code.
- To support nullable reference type annotations in generic APIs.
- To distinguish non-nullable generics from nullable ones.

---

### Syntax Rules and Structure

```
where T : notnull
```

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The type parameter.
- `:` — Separator.
- `notnull` — Requires a non-nullable type.

#### Syntax Rules

- `notnull` is a primary constraint; it must appear first.
- Mutually exclusive with `class`, `class?`, `struct`, `unmanaged`, and `default`.
- Works with both reference and value types (as long as they are non-nullable).

#### Constraints and Limitations

- Cannot be combined with other primary constraints.
- In a nullable context, `notnull` ensures `T` is not `T?`.

---

### Annotated Code Examples and Expected Outputs

#### Example: `notnull` Constraint

```csharp
using System;

public class NonNullContainer<T> where T : notnull
{
    public T Item { get; set; } = default!;
}

class Program
{
    static void Main()
    {
        var stringContainer = new NonNullContainer<string> { Item = "Hello" };
        Console.WriteLine(stringContainer.Item);   // Hello

        var intContainer = new NonNullContainer<int> { Item = 42 };
        Console.WriteLine(intContainer.Item);      // 42
    }
}
```

**Expected Output**

```
Hello
42
```

**Why this output occurs**

- Both `string` (non-nullable reference type) and `int` (non-nullable value type) satisfy `notnull`.

---

### Real-World Cases

- **Nullable-aware APIs** — Ensuring generic arguments are non-nullable.
- **Dictionaries with non-null keys** — `where TKey : notnull`.
- **Configuration containers** — Non-nullable values.

---

### References

- Constraints on type parameters — `notnull` constraint - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#notnull-constraint

---

## 4. Unmanaged Constraint (`unmanaged`)

### Definitions

**Core Definition**
The `unmanaged` constraint requires that the type argument be a **non-nullable unmanaged type**—a type whose values do not contain references and can be used in unsafe pointer operations.

**Technical Definition**
`where T : unmanaged` specifies that the type argument must be a non-nullable unmanaged type. The `unmanaged` constraint **implies the `struct` constraint** and **cannot be combined with `struct` or `new()`**. Unmanaged types include primitive numeric types, `bool`, `char`, enums, and structs composed entirely of unmanaged types. The constraint enables the use of pointers and `sizeof` in generic code.

**Beginner-Friendly Explanation**
`unmanaged` means **“T must be a value type with no references”** —something that can live entirely on the stack and be manipulated with pointers.

---

### Purposes

- To ensure that `T` can be used in unsafe code and interop scenarios.
- To enable pointer operations and memory-size calculations in generic algorithms.
- To restrict generics to types with no managed references.

---

### Syntax Rules and Structure

```
where T : unmanaged
```

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The type parameter.
- `:` — Separator.
- `unmanaged` — Requires an unmanaged type.

#### Syntax Rules

- `unmanaged` is a primary constraint; it must appear first.
- Implies `struct`; cannot be combined with `struct` or `new()`.
- Mutually exclusive with `class`, `class?`, `notnull`, and `default`.

#### Constraints and Limitations

- Cannot be combined with `struct` (redundant).
- Cannot be combined with `new()` (redundant via `struct`).
- Does not allow reference types or structs containing references.

---

### Annotated Code Examples and Expected Outputs

#### Example: `unmanaged` Constraint

```csharp
using System;

public unsafe class UnmanagedContainer<T> where T : unmanaged
{
    public T Value;

    public int Size() => sizeof(T);
}

class Program
{
    static void Main()
    {
        var intContainer = new UnmanagedContainer<int> { Value = 42 };
        Console.WriteLine(intContainer.Size());   // 4
    }
}
```

**Expected Output**

```
4
```

**Why this output occurs**

- `int` is an unmanaged type; `sizeof(int)` is 4.

---

### Real-World Cases

- **Interop and P/Invoke** — Generic wrappers for unmanaged types.
- **Low-level memory operations** — `sizeof` and pointer arithmetic.
- **Serialization** — Fixed-layout binary data.

---

### References

- Constraints on type parameters — `unmanaged` constraint - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#unmanaged-constraint

---

## 5. Constructor Constraint (`new()`)

### Definitions

**Core Definition**
The `new()` constraint requires that the type argument have a **public parameterless constructor**. When used with other constraints, it must appear **last**.

**Technical Definition**
`where T : new()` specifies that the type argument must have a public parameterless constructor. The `new()` constraint makes it possible for generic code to instantiate `T` using `new T()`. When used together with other constraints, the `new()` constraint must be specified **last**. The `new()` constraint **cannot be combined with `struct` or `unmanaged`** because those constraints already imply a parameterless constructor.

**Beginner-Friendly Explanation**
`new()` means **“T must have a parameterless constructor”** —you can call `new T()` inside your generic code.

---

### Purposes

- To enable generic factories that create instances of `T`.
- To support object creation in generic algorithms.
- To enforce a parameterless-constructor contract.

---

### Syntax Rules and Structure

```
where T : new()
where T : IInterface, new()
```

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The type parameter.
- `:` — Separator.
- `new()` — Requires a public parameterless constructor.

#### Syntax Rules

- `new()` must appear last in the constraint list.
- Cannot be combined with `struct` or `unmanaged`.
- Can be combined with `class`, `class?`, `notnull`, base class, and interface constraints.

#### Constraints and Limitations

- Does not work with abstract types (abstract types cannot be instantiated).
- Does not work with types that have only parameterized constructors.

---

### Annotated Code Examples and Expected Outputs

#### Example: `new()` Constraint

```csharp
using System;

public class Factory<T> where T : new()
{
    public T Create() => new T();
}

public class Widget
{
    public string Name { get; set; } = "Default";
}

class Program
{
    static void Main()
    {
        var factory = new Factory<Widget>();
        var widget = factory.Create();
        Console.WriteLine(widget.Name);   // Default
    }
}
```

**Expected Output**

```
Default
```

**Why this output occurs**

- `Widget` has a public parameterless constructor, satisfying `new()`.

---

### Real-World Cases

- **Generic factories** — `Activator.CreateInstance<T>()` alternatives.
- **Dependency injection** — Creating instances of registered types.
- **Test data builders** — Generic object creation.

---

### References

- Constraints on type parameters — `new()` constraint - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#constructor-constraint

---

## 6. Base Class Constraint

### Definitions

**Core Definition**
The base class constraint requires that the type argument be **derived from or be the specified base class**.

**Technical Definition**
`where T : BaseClass` specifies that the type argument must be or derive from the specified base class. The base class constraint must appear **before interface constraints** in the constraint list. In a nullable context, `T` must be a non-nullable reference type derived from the specified base class; `where T : BaseClass?` allows nullable or non-nullable types derived from the base class.

**Beginner-Friendly Explanation**
The base class constraint means **“T must be this class or a subclass of it.”**

---

### Purposes

- To ensure that `T` shares a common base class, enabling access to base-class members.
- To restrict generic APIs to a specific class hierarchy.
- To enable polymorphic operations on `T`.

---

### Syntax Rules and Structure

```
where T : BaseClass
where T : BaseClass?
```

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The type parameter.
- `:` — Separator.
- `BaseClass` — The required base class.

#### Syntax Rules

- The base class constraint must appear before interface constraints.
- Cannot be combined with `struct` or `unmanaged` (they are value types; a base class is a reference type).
- Cannot use `System.Object`, `System.Array`, or `System.ValueType` as a base class constraint.

#### Constraints and Limitations

- Cannot combine with `struct` or `unmanaged`.
- The base class must be accessible to the generic type.

---

### Annotated Code Examples and Expected Outputs

#### Example: Base Class Constraint

```csharp
using System;

public class Animal
{
    public string Name { get; set; } = "Unknown";
}

public class AnimalContainer<T> where T : Animal
{
    public T? Item { get; set; }

    public void PrintName() => Console.WriteLine(Item?.Name);
}

public class Dog : Animal { public Dog() { Name = "Rex"; } }

class Program
{
    static void Main()
    {
        var container = new AnimalContainer<Dog> { Item = new Dog() };
        container.PrintName();   // Rex
    }
}
```

**Expected Output**

```
Rex
```

**Why this output occurs**

- `Dog` derives from `Animal`, satisfying the base class constraint.

---

### Real-World Cases

- **Entity repositories** — `where T : EntityBase`.
- **UI controls** — `where T : Control`.
- **Domain hierarchies** — Generic operations on derived types.

---

### References

- Constraints on type parameters — Base class constraint - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#base-class-constraint

---

## 7. Interface Constraint

### Definitions

**Core Definition**
The interface constraint requires that the type argument **be or implement the specified interface**. Multiple interface constraints can be specified.

**Technical Definition**
`where T : IInterface` specifies that the type argument must be or implement the specified interface. Multiple interface constraints can be specified. The constraining interface can also be generic. In a nullable context, `T` must be a non-nullable type that implements the specified interface; `where T : IInterface?` allows nullable reference types, non-nullable reference types, and value types (but not nullable value types).

**Beginner-Friendly Explanation**
The interface constraint means **“T must implement this interface”** —you can call the interface’s methods on `T`.

---

### Purposes

- To ensure that `T` implements a specific contract.
- To enable access to interface members in generic code.
- To support multiple interface constraints for richer contracts.
- To enable generic algorithms that rely on interface contracts.

---

### Syntax Rules and Structure

```
where T : IInterface
where T : IInterface1, IInterface2
```

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The type parameter.
- `:` — Separator.
- `IInterface` — The required interface.

#### Syntax Rules

- Multiple interface constraints are allowed.
- Interface constraints can be generic (e.g., `IComparable<T>`).
- Interface constraints appear after the base class constraint.
- `IInterface?` allows nullable types (C# 8+).

#### Constraints and Limitations

- Cannot use marker interfaces as the sole constraint without good reason.
- The interface must be accessible.

---

### Annotated Code Examples and Expected Outputs

#### Example: Interface Constraint

```csharp
using System;

public interface IIdentifiable
{
    int Id { get; }
}

public class Entity : IIdentifiable
{
    public int Id { get; set; }
}

public class Repository<T> where T : IIdentifiable
{
    public void PrintId(T entity) => Console.WriteLine(entity.Id);
}

class Program
{
    static void Main()
    {
        var repo = new Repository<Entity>();
        repo.PrintId(new Entity { Id = 42 });   // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- `Entity` implements `IIdentifiable`, so the interface constraint is satisfied and `Id` is accessible.

---

### Real-World Cases

- **`IComparable<T>`** — Sorting and comparison.
- **`IEquatable<T>`** — Equality checks.
- **`IRepository<T>`** — Data access contracts.
- **`INumber<T>`** — Generic math (C# 11+).

---

### References

- Constraints on type parameters — Interface constraint - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#interface-constraint

---

## 8. Type Parameter Constraint (`where T : U`)

### Definitions

**Core Definition**
The type parameter constraint requires that the type argument for `T` be **derived from or be the type argument for `U`**.

**Technical Definition**
`where T : U` specifies that the type argument supplied for `T` must be or derive from the argument supplied for `U`. In a nullable context, if `U` is a non-nullable reference type, `T` must be a non-nullable reference type; if `U` is a nullable reference type, `T` can be nullable or non-nullable.

**Beginner-Friendly Explanation**
`where T : U` means **“T must be U or a subtype of U.”** It is a constraint between two type parameters.

---

### Purposes

- To enforce a relationship between two type parameters.
- To ensure that one type parameter is a subtype of another.
- To enable generic algorithms that depend on type parameter relationships.

---

### Syntax Rules and Structure

```
where T : U
```

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The constrained type parameter.
- `:` — Separator.
- `U` — The constraining type parameter.

#### Syntax Rules

- `U` must be a type parameter of the same generic type or method.
- `T` must be or derive from `U`.
- Cannot be combined with `struct` or `unmanaged` if `U` is a reference type.

#### Constraints and Limitations

- The relationship must be satisfiable by all valid type arguments.
- Cannot create circular constraints.

---

### Annotated Code Examples and Expected Outputs

#### Example: Type Parameter Constraint

```csharp
using System;

public class Base { public void Hello() => Console.WriteLine("Base"); }
public class Derived : Base { }

public class Container<T, U> where T : U
{
    public void Show(T item) => Console.WriteLine(item);
}

class Program
{
    static void Main()
    {
        var container = new Container<Derived, Base>();
        container.Show(new Derived());   // Derived (ToString)
    }
}
```

**Expected Output**

```
Derived
```

**Why this output occurs**

- `Derived` derives from `Base`, satisfying `where T : U`.

---

### Real-World Cases

- **Layered architectures** — Ensuring derived types satisfy base constraints.
- **Generic hierarchies** — Relationships between type parameters.
- **Covariant-like patterns** — Enforcing subtype relationships.

---

### References

- Constraints on type parameters — Type parameter constraint - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters

---

## 9. Multiple Constraints

### Definitions

**Core Definition**
Multiple constraints combine several restrictions on a single type parameter, refining the set of valid type arguments and enabling access to a richer set of members.

**Technical Definition**
A type parameter can have **multiple constraints** specified in a single `where` clause. The constraints must appear in a **specific order**: primary constraints (`class`, `class?`, `struct`, `unmanaged`, `notnull`, or `default`) come first, followed by a **base class constraint**, then **interface constraints**, and finally the **`new()` constraint**. Some constraints are **mutually exclusive** (e.g., `class` and `struct`, `struct` and `unmanaged`, `class` and a specific base class). All constraints for a single type parameter must appear in a **single `where` clause**; multiple `where` clauses for the same type parameter are invalid.

**Beginner-Friendly Explanation**
Multiple constraints let you say **“T must be a class, implement `IComparable<T>`, and have a parameterless constructor”** all in one rule.

---

### Purposes

- To refine the set of valid type arguments precisely.
- To enable access to multiple capabilities of `T`.
- To express rich API contracts.
- To prevent invalid type arguments at compile time.

---

### Syntax Rules and Structure

```
where T : class, IComparable<T>, new()
where T : struct, IEquatable<T>
where T : BaseClass, IInterface1, IInterface2, new()
```

**Component breakdown**

- **Primary constraint** — First (e.g., `class`, `struct`).
- **Base class** — After primary.
- **Interfaces** — After base class.
- **`new()`** — Last.

#### Syntax Rules

- Order: primary → base class → interfaces → `new()`.
- Mutually exclusive constraints cannot be combined (e.g., `class` + `struct`).
- `struct` implies `new()`, so `struct` + `new()` is invalid.
- `unmanaged` implies `struct`, so `unmanaged` + `struct` and `unmanaged` + `new()` are invalid.
- All constraints for one type parameter must be in one `where` clause.

#### Constraints and Limitations

- Cannot use multiple `where` clauses for the same type parameter.
- Invalid combinations produce compile-time errors (CS0401, CS0406, CS0409, CS0449, CS0450, CS0451).

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Constraints

```csharp
using System;

public class Factory<T> where T : class, IComparable<T>, new()
{
    public T Create() => new T();
    public bool IsGreater(T a, T b) => a.CompareTo(b) > 0;
}

public class Widget : IComparable<Widget>
{
    public int Value { get; set; }
    public int CompareTo(Widget? other) => Value.CompareTo(other?.Value ?? 0);
}

class Program
{
    static void Main()
    {
        var factory = new Factory<Widget>();
        var w1 = factory.Create(); w1.Value = 10;
        var w2 = factory.Create(); w2.Value = 5;
        Console.WriteLine(factory.IsGreater(w1, w2));   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `Widget` satisfies all three constraints: it is a `class`, implements `IComparable<Widget>`, and has a parameterless constructor.

---

### Real-World Cases

- **Generic repositories** — `where T : class, IEntity, new()`.
- **Generic algorithms** — `where T : IComparable<T>, IEquatable<T>`.
- **Factory patterns** — `where T : class, new()`.

---

### References

- Resolve errors related to generic type parameters — Constraint ordering - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/generic-type-parameters-errors
- Constraints on type parameters — Multiple constraints - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters

---

## 10. Constraint Best Practices

### Definitions

**Core Definition**
Constraint best practices are the design guidelines that ensure generic constraints are **meaningful, minimal, correctly ordered, and compatible** with API evolution and code analysis rules.

**Technical Definition**
Best practices for constraints derive from the Framework Design Guidelines and code analysis rules. Key recommendations include: **apply the least restrictive constraints necessary** to enable the required operations; **order constraints correctly** (primary → base class → interfaces → `new()`); **avoid mutually exclusive or redundant constraints**; **use `notnull` for nullable-aware APIs**; **use `unmanaged` only for interop and unsafe scenarios**; **prefer interface constraints over base class constraints** when the relationship is contract-based; and **do not declare static members on generic types** (CA1000) or use excessive type parameters (CA1005). Constraints should **document the API contract** and **enforce type safety** without over-restricting callers.

**Beginner-Friendly Explanation**
Best practices for constraints are simple: **use only the constraints you need**, **put them in the right order**, **do not use contradictory constraints**, and **document what the constraints require**.

---

### Purposes

- To ensure constraints are correct and meaningful.
- To avoid compile-time errors and confusion.
- To keep generic APIs flexible and usable.
- To align with .NET design conventions and analyzers.
- To document the capabilities required of type arguments.

---

### Guidelines and Structure

#### General Guidelines

```
// Use the least restrictive constraints necessary
public T Max<T>(T a, T b) where T : IComparable<T> => ...;

// Order constraints correctly
where T : class, IComparable<T>, new()

// Avoid mutually exclusive constraints
// (class + struct, struct + new(), unmanaged + struct, etc.)

// Use notnull for nullable-aware APIs
where T : notnull

// Prefer interface constraints for contract-based relationships
where T : IEquatable<T>

// Avoid static members on generic types (CA1000)
// Avoid excessive type parameters (CA1005) — ≤ 2 for public types
```

**Component breakdown**

- **Minimal constraints** — Use only what is needed.
- **Correct order** — Primary → base class → interfaces → `new()`.
- **No contradictions** — Do not combine mutually exclusive constraints.
- **`notnull`** — For nullable-aware non-nullable types.
- **Interface preference** — Contract-based constraints.
- **CA1000/CA1005** — Avoid static members on generic types and excessive type parameters.

#### Rules and Constraints

- **Do** apply the least restrictive constraints that enable the required operations.
- **Do** order constraints correctly (primary → base class → interfaces → `new()`).
- **Do** use `notnull` for nullable-aware APIs.
- **Do** document the constraints in XML comments.
- **Do not** combine mutually exclusive constraints.
- **Do not** declare static members on generic types (CA1000).
- **Do not** use more than two type parameters on externally visible generic types (CA1005).

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Constraints

```csharp
using System;

public static class CollectionHelper
{
    // Minimal, correctly ordered constraints
    public static T Max<T>(T a, T b) where T : IComparable<T>
        => a.CompareTo(b) >= 0 ? a : b;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(CollectionHelper.Max(10, 20));    // 20
        Console.WriteLine(CollectionHelper.Max("A", "B"));  // B
    }
}
```

**Expected Output**

```
20
B
```

**Why this output occurs**

- The `IComparable<T>` constraint enables `CompareTo` on `T`; the constraint is minimal and correctly ordered.

---

### Real-World Cases

- **Generic math** — `where T : INumber<T>` (C# 11+).
- **Repository patterns** — `where T : class, IEntity`.
- **Factory methods** — `where T : new()`.

---

### References

- Constraints on type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- CA1000: Do not declare static members on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1000
- CA1005: Avoid excessive parameters on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1005
- Generic math — .NET - https://learn.microsoft.com/en-us/dotnet/standard/generics/math

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `class`, `struct`, `new()` constraints | Core since **C# 2.0**; stable. |
| `notnull` constraint | Introduced in **C# 8.0**; version-specific. |
| `unmanaged` constraint | Introduced in **C# 7.3**; version-specific. |
| `class?` and `IInterface?` constraints | Introduced in **C# 8.0** (nullable context). |
| `default` constraint | Introduced in **C# 9.0**; for overrides and explicit interface implementations. |
| Generic math (`INumber<T>`, static abstract members) | Introduced in **C# 11 / .NET 7**; version-specific. |
| `delegate` constraint | **Not valid**; use `System.Delegate` instead (CS9011). |
| `System.Object`, `System.Array`, `System.ValueType` as base class constraints | **Not allowed** (CS0702). |
| Multiple `where` clauses for the same type parameter | **Invalid** (CS0409). |

---

## References (Consolidated)

- Constraints on type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- where (generic type constraint) — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint
- Resolve errors related to generic type parameters — Constraint ordering - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/generic-type-parameters-errors
- Generic math — .NET - https://learn.microsoft.com/en-us/dotnet/standard/generics/math
- CA1000: Do not declare static members on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1000
- CA1005: Avoid excessive parameters on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1005
- Classes — C# language specification (type parameter constraints) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes

---