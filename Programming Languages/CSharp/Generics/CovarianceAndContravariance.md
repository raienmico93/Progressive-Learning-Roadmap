# C# Covariance and Contravariance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Covariance and contravariance are **variance** features in C# that allow generic types, interfaces, and delegates to be assigned more flexibly than their type parameters strictly dictate. **Covariance** (`out`) allows a more derived type to be used where a less derived type is expected—typically for **return values**. **Contravariance** (`in`) allows a less derived type to be used where a more derived type is expected—typically for **input parameters**. Together, they make generic code more reusable and flexible while preserving compile-time type safety.

**Technical Definition**
Variance is a C# concept that allows a type to be used as a more derived type than specified in a delegate or interface. **Covariance** allows a method to have a return type that is more derived than the type defined in the delegate or interface method. **Contravariance** allows a method to accept parameters that are less derived than the parameters of the delegate or interface type. Variance support includes generic and non-generic delegates. A generic interface that has covariant or contravariant generic type parameters is called a **variant interface**. The `out` keyword marks a type parameter as covariant, and `in` marks it as contravariant. **Reference types** support variance; **value types do not**. `ref`, `in`, and `out` parameters in C# cannot be variant.

**Beginner-Friendly Explanation**
Covariance and contravariance are about **flexibility in what types you can plug into generic containers and delegates**. Think of a hierarchy: `Animal` → `Dog`. **Covariance** (`out`) lets you treat a `Dog` producer as an `Animal` producer: a `Func<Dog>` can be assigned to a `Func<Animal>` because producing a `Dog` also counts as producing an `Animal`. **Contravariance** (`in`) goes the other way: a `Action<Animal>` (something that can accept any `Animal`) can be assigned to an `Action<Dog>` because anything that can handle an `Animal` can certainly handle a `Dog`. The keywords `out` and `in` are your way of telling the compiler “this type parameter only appears in output positions” (covariant) or “only in input positions” (contravariant), and then the compiler lets you assign more derived or less derived types accordingly.

---

### Key Characteristics

- **Two variance kinds** — Covariance (`out`, derived → base for outputs) and contravariance (`in`, base → derived for inputs).
- **`out` for outputs** — A covariant type parameter can be used **only** as a return type of interface methods or delegate return types.
- **`in` for inputs** — A contravariant type parameter can be used **only** as a parameter type of interface methods or delegate parameters.
- **Reference types only** — Variance works for reference types; value types do not support variance.
- **`ref` / `in` / `out` parameters cannot be variant** — Ref-style parameters block variance.
- **Variant interfaces and delegates** — `IEnumerable<out T>`, `Func<out TResult>`, `Action<in T>`, and `IComparer<in T>` are standard examples.
- **Delegate binding variance** — Works with all delegate types, not just generic delegates with variant parameters.
- **Combined effects** — Variance in delegate binding and variance in generic type parameters can combine for powerful flexibility.

---

### Prerequisites

- Understanding of C# generics (type parameters, generic interfaces, delegates).
- Familiarity with inheritance and type hierarchies (base class vs. derived class).
- Knowledge of `Func<T>`, `Action<T>`, and `IEnumerable<T>`.
- Awareness of reference types vs. value types.
- Basic understanding of method signatures (parameters and return types).

---

### Related Programming Areas

- **LINQ** — `IEnumerable<out T>` is covariant, enabling `IEnumerable<Derived>` to be used as `IEnumerable<Base>`.
- **Delegates and events** — `Func<out TResult>` and `Action<in T>` rely on variance.
- **Generic interfaces** — `IComparer<in T>`, `IEqualityComparer<in T>`.
- **Collections** — Variance allows collections of derived types to be consumed as collections of base types.
- **API design** — Variance annotations (`out`/`in`) shape the flexibility of generic APIs.

---

### Core Concepts / Features

1. Covariance (`out`)
2. Contravariance (`in`)
3. Variance in Generics
4. Variance in Delegates
5. Variance Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Covariance (`out`)

### Definitions

**Core Definition**
Covariance (`out`) allows a generic type or delegate to be assigned a **more derived** type argument than its declared type parameter, enabling derived types to be used where base types are expected—primarily for **return values**.

**Technical Definition**
Covariance allows interface methods to have more derived return types than that defined by the generic type parameters. A covariant type parameter is marked with the `out` keyword. The covariant type must satisfy these conditions: the type is used **only as a return type** of interface methods and **not** used as a type of method arguments; the type is **not** used as a generic constraint for interface methods. There is one exception: if a contravariant generic delegate is a method parameter, a covariant type parameter of the interface type can be used to specify a contravariant type parameter of the delegate type. Covariance applies to reference types; value types do not support variance.

**Beginner-Friendly Explanation**
`out` means **“this type only comes out.”** If a component produces `Dog`, it can be used wherever a component producing `Animal` is needed, because a `Dog` **is an** `Animal`.

---

### Purposes

- To enable **`IEnumerable<Derived>` → `IEnumerable<Base>`** assignments.
- To allow generic producers to be used more flexibly.
- To support **covariant return types** in delegates.
- To make LINQ and collection APIs more usable.
- To preserve type safety while increasing flexibility.

---

### Syntax Rules and Structure

```
// Covariant interface
interface ICovariant<out R>
{
    R GetSomething();          // OK: R used only as return type
    // void SetSomething(R arg); // Compiler error: R used as parameter
}

// Covariant delegate
delegate R Producer<out R>();

// Usage
ICovariant<Dog> dogProvider = ...;
ICovariant<Animal> animalProvider = dogProvider;   // covariance
```

**Component breakdown**

- `out R` — Declares `R` as covariant.
- `R GetSomething()` — `R` in a return position (allowed).
- `void SetSomething(R arg)` — `R` in a parameter position (not allowed for covariance).

#### Syntax Rules

- `out` appears before the type parameter name in the declaration.
- The covariant type can be used **only** as a return type of interface methods or delegate return types.
- The covariant type **cannot** be used as a method parameter (with a narrow exception for contravariant delegates).
- The covariant type **cannot** be used as a generic constraint for interface methods.
- Reference types support covariance; **value types do not**.

#### Constraints and Limitations

- `ref`, `in`, and `out` parameters cannot be variant.
- Covariant type parameters cannot be used in `ref` or `out` parameter positions.
- Covariance does not apply to value types.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Covariant `IEnumerable<T>`

```csharp
using System;
using System.Collections.Generic;

class Animal { public string Name = "Animal"; }
class Dog : Animal { public Dog() { Name = "Dog"; } }

class Program
{
    static void Main()
    {
        IEnumerable<Dog> dogs = new List<Dog> { new Dog() };

        // Covariance: IEnumerable<Dog> → IEnumerable<Animal>
        IEnumerable<Animal> animals = dogs;

        foreach (Animal a in animals)
            Console.WriteLine(a.Name);   // Dog
    }
}
```

**Expected Output**

```
Dog
```

**Why this output occurs**

- `IEnumerable<out T>` is covariant; `IEnumerable<Dog>` can be assigned to `IEnumerable<Animal>` because `Dog` derives from `Animal`.

---

#### Example 2: Covariant `Func<TResult>` Delegate

```csharp
using System;

class Person { }
class Employee : Person { }

class Program
{
    static Employee FindEmployee(string title) => new Employee();

    static void Main()
    {
        // Covariance: Func<string, Employee> → Func<string, Person>
        Func<string, Person> func = FindEmployee;

        Person person = func("Manager");
        Console.WriteLine(person is Employee);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `Func<out TResult>` is covariant in `TResult`; a method returning `Employee` can be assigned to a delegate expecting `Person`.

---

#### Example 3: Covariant Interface with Default Implementation

```csharp
using System;

interface IProducer<out T>
{
    T Produce();
}

class DogProducer : IProducer<Dog>
{
    public Dog Produce() => new Dog();
}

class Animal { }
class Dog : Animal { }

class Program
{
    static void Main()
    {
        IProducer<Dog> dogProducer = new DogProducer();
        IProducer<Animal> animalProducer = dogProducer;   // covariance
        Animal a = animalProducer.Produce();
        Console.WriteLine(a is Dog);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `IProducer<out T>` is covariant because `T` is used only in return position; `IProducer<Dog>` can be assigned to `IProducer<Animal>`.

---

### Real-World Cases

- **LINQ** — `IEnumerable<Derived>` can be queried as `IEnumerable<Base>`.
- **Factory methods** — Covariant `Func<TResult>` delegates.
- **Collection APIs** — Returning `IEnumerable<Base>` from methods that produce derived types.

---

### References

- Covariance and Contravariance (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/
- Creating Variant Generic Interfaces (C#) — Covariance - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/creating-variant-generic-interfaces
- Variance in Delegates (C#) — Covariance - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-delegates

---

## 2. Contravariance (`in`)

### Definitions

**Core Definition**
Contravariance (`in`) allows a generic type or delegate to be assigned a **less derived** type argument than its declared type parameter, enabling base types to be used where derived types are expected—primarily for **input parameters**.

**Technical Definition**
Contravariance allows interface methods to have argument types that are less derived than that specified by the generic parameters. A contravariant type parameter is marked with the `in` keyword. The contravariant type can be used **only as a type of method arguments** and **not** as a return type of interface methods. The contravariant type can also be used for **generic constraints**. Contravariance applies to reference types; value types do not support variance. `ref`, `in`, and `out` parameters cannot be variant.

**Beginner-Friendly Explanation**
`in` means **“this type only comes in.”** If a component can accept any `Animal`, it can certainly be used where a component accepting a `Dog` is needed, because a `Dog` **is an** `Animal`.

---

### Purposes

- To enable **`IComparer<Base>` → `IComparer<Derived>`** assignments.
- To allow generic consumers to be used more flexibly.
- To support **contravariant parameter types** in delegates.
- To make comparison and equality APIs more usable.
- To preserve type safety while increasing flexibility.

---

### Syntax Rules and Structure

```
// Contravariant interface
interface IContravariant<in A>
{
    void SetSomething(A sampleArg);    // OK: A used only as parameter
    // A GetSomething();                // Compiler error: A used as return type
}

// Contravariant delegate
delegate void Consumer<in A>(A argument);

// Usage
IContravariant<Animal> animalConsumer = ...;
IContravariant<Dog> dogConsumer = animalConsumer;   // contravariance
```

**Component breakdown**

- `in A` — Declares `A` as contravariant.
- `void SetSomething(A sampleArg)` — `A` in a parameter position (allowed).
- `A GetSomething()` — `A` in a return position (not allowed for contravariance).

#### Syntax Rules

- `in` appears before the type parameter name in the declaration.
- The contravariant type can be used **only** as a type of method arguments.
- The contravariant type **cannot** be used as a return type of interface methods.
- The contravariant type **can** be used for generic constraints.
- Reference types support contravariance; **value types do not**.

#### Constraints and Limitations

- `ref`, `in`, and `out` parameters cannot be variant.
- Contravariant type parameters cannot be used in return positions.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Contravariant `IComparer<T>`

```csharp
using System;
using System.Collections.Generic;

class Animal { public string Name = "Animal"; }
class Dog : Animal { public Dog() { Name = "Dog"; } }

class AnimalComparer : IComparer<Animal>
{
    public int Compare(Animal? x, Animal? y) =>
        string.Compare(x?.Name, y?.Name);
}

class Program
{
    static void Main()
    {
        IComparer<Animal> animalComparer = new AnimalComparer();

        // Contravariance: IComparer<Animal> → IComparer<Dog>
        IComparer<Dog> dogComparer = animalComparer;

        Console.WriteLine(dogComparer.Compare(new Dog(), new Dog()));   // 0
    }
}
```

**Expected Output**

```
0
```

**Why this output occurs**

- `IComparer<in T>` is contravariant; an `IComparer<Animal>` can be used as `IComparer<Dog>` because anything that can compare `Animal`s can compare `Dog`s.

---

#### Example 2: Contravariant `Action<T>` Delegate

```csharp
using System;

class Animal { public string Name = "Animal"; }
class Dog : Animal { public Dog() { Name = "Dog"; } }

class Program
{
    static void HandleAnimal(Animal animal) =>
        Console.WriteLine($"Handling {animal.Name}");

    static void Main()
    {
        // Contravariance: Action<Animal> → Action<Dog>
        Action<Dog> dogAction = HandleAnimal;

        dogAction(new Dog());   // Handling Dog
    }
}
```

**Expected Output**

```
Handling Dog
```

**Why this output occurs**

- `Action<in T>` is contravariant; a method accepting `Animal` can be assigned to `Action<Dog>` because `Dog` is an `Animal`.

---

#### Example 3: Contravariant Interface with Generic Constraint

```csharp
using System;

interface IHandler<in T>
{
    void Handle(T item);
    void HandleGeneric<U>() where U : T;   // contravariant type as constraint
}

class AnimalHandler : IHandler<Animal>
{
    public void Handle(Animal item) => Console.WriteLine("Handling animal");
    public void HandleGeneric<U>() where U : Animal => Console.WriteLine("Generic handled");
}

class Animal { }
class Dog : Animal { }

class Program
{
    static void Main()
    {
        IHandler<Animal> animalHandler = new AnimalHandler();
        IHandler<Dog> dogHandler = animalHandler;   // contravariance

        dogHandler.Handle(new Dog());   // Handling animal
    }
}
```

**Expected Output**

```
Handling animal
```

**Why this output occurs**

- `IHandler<in T>` is contravariant; `IHandler<Animal>` can be used as `IHandler<Dog>`. The generic constraint `where U : T` is allowed for contravariant type parameters.

---

### Real-World Cases

- **Comparison** — `IComparer<in T>` allows comparers for base types to be used for derived types.
- **Equality** — `IEqualityComparer<in T>` enables equality checks with base-type comparisons.
- **Event handlers** — `Action<in T>` for handling derived event args with base handlers.
- **Dependency injection** — Contravariant service interfaces.

---

### References

- Covariance and Contravariance (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/
- Creating Variant Generic Interfaces (C#) — Contravariance - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/creating-variant-generic-interfaces
- Variance in Delegates (C#) — Contravariance - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-delegates

---

## 3. Variance in Generics

### Definitions

**Core Definition**
Variance in generics is the ability to assign a generic type with one type argument to a variable of the same generic type with a different (more or less derived) type argument, enabled by the `out` and `in` annotations on the generic type parameters.

**Technical Definition**
You can declare generic type parameters in interfaces as covariant or contravariant. Covariance allows interface methods to have more derived return types than those defined by the generic type parameters; contravariance allows interface methods to have argument types that are less derived than those specified by the generic parameters. A generic interface with covariant or contravariant generic type parameters is called a **variant interface**. Variant generic interfaces are declared by using the `in` and `out` keywords for generic type parameters. **Value types do not support variance**, and `ref`, `in`, and `out` parameters cannot be variant. The .NET Framework 4 introduced variance support for several existing generic interfaces, including `IEnumerable<out T>`, `IComparer<in T>`, `IEqualityComparer<in T>`, `IQueryable<out T>`, and `IOrderedQueryable<out T>`.

**Beginner-Friendly Explanation**
Variance in generics means **you can use a `List<Dog>` where a `List<Animal>` is expected, or a `IComparer<Animal>` where an `IComparer<Dog>` is expected**, provided the generic type is declared with `out` or `in` appropriately.

---

### Purposes

- To make generic interfaces **more flexible** and **reusable**.
- To enable **covariant collections** (`IEnumerable<out T>`).
- To enable **contravariant comparers** (`IComparer<in T>`).
- To allow **derived types to be consumed as base types** (covariance).
- To allow **base-type handlers to be used for derived types** (contravariance).

---

### Syntax Rules and Structure (Variant Interface Rules)

| Position | Covariant (`out`) | Contravariant (`in`) |
|---|---|---|
| **Return type** | ✔️ Allowed | ❌ Not allowed |
| **Parameter type** | ❌ Not allowed (except contravariant delegate) | ✔️ Allowed |
| **Generic constraint** | ❌ Not allowed | ✔️ Allowed |
| **`ref` / `in` / `out` parameter** | ❌ Not allowed | ❌ Not allowed |
| **Value types** | ❌ Not supported | ❌ Not supported |

**Component breakdown**

- **`out`** — Covariant; used only in output positions.
- **`in`** — Contravariant; used only in input positions.
- **`ref` / `in` / `out`** — These parameters block variance.

#### Syntax Rules

- Variant interfaces use `in` and `out` on type parameters.
- Both covariance and contravariance can be supported in the same interface for different type parameters.
- Implementing classes use the same syntax as non-variant interfaces.

#### Constraints and Limitations

- **Value types do not support variance** — `IEnumerable<int>` cannot be assigned to `IEnumerable<object>`.
- `ref`, `in`, and `out` parameters cannot be variant.
- Covariant types cannot be used as generic constraints for interface methods.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Combining Covariance and Contravariance

```csharp
using System;

interface IVariant<out R, in A>
{
    R GetSomething();
    void SetSomething(A sampleArg);
    R GetSetSomethings(A sampleArg);
}

class Animal { }
class Dog : Animal { }

class VariantImplementation : IVariant<Dog, Animal>
{
    public Dog GetSomething() => new Dog();
    public void SetSomething(Animal sampleArg) { }
    public Dog GetSetSomethings(Animal sampleArg) => new Dog();
}

class Program
{
    static void Main()
    {
        IVariant<Dog, Animal> impl = new VariantImplementation();

        // Covariance: Dog → Animal in R
        IVariant<Animal, Animal> covariant = impl;

        // Contravariance: Animal → Dog in A
        IVariant<Dog, Dog> contravariant = impl;

        Console.WriteLine(covariant.GetSomething() is Animal);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `IVariant<out R, in A>` supports both covariance in `R` and contravariance in `A`; assignments are valid in both directions.

---

#### Example 2: List of Derived Types as IEnumerable of Base

```csharp
using System;
using System.Collections.Generic;

class Animal { public string Name = "Animal"; }
class Dog : Animal { public Dog() { Name = "Dog"; } }

class Program
{
    static void PrintAll(IEnumerable<Animal> animals)
    {
        foreach (Animal a in animals)
            Console.WriteLine(a.Name);
    }

    static void Main()
    {
        List<Dog> dogs = new List<Dog> { new Dog() };
        PrintAll(dogs);   // covariance: List<Dog> → IEnumerable<Animal>
    }
}
```

**Expected Output**

```
Dog
```

**Why this output occurs**

- `IEnumerable<out T>` is covariant; `List<Dog>` implements `IEnumerable<Dog>`, which is assignable to `IEnumerable<Animal>`.

---

### Real-World Cases

- **LINQ queries** — `IEnumerable<Derived>` consumed as `IEnumerable<Base>`.
- **Comparison** — `IComparer<Animal>` used as `IComparer<Dog>`.
- **API design** — Variant interfaces (`out`/`in`) improve flexibility.

---

### References

- Creating Variant Generic Interfaces (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/creating-variant-generic-interfaces
- Variance in Generic Interfaces (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-generic-interfaces
- Covariance and Contravariance (C#) — Overview - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/

---

## 4. Variance in Delegates

### Definitions

**Core Definition**
Variance in delegates allows a delegate to be assigned a method whose signature does not exactly match the delegate type, provided the method’s parameter types and return type are compatible under covariance and contravariance rules.

**Technical Definition**
Variance in C# allows you to use more flexible method signatures with delegates. It enables a method to be assigned to a delegate even if their parameter types or return types are not exactly the same, as long as they follow certain rules. **Covariance** permits a method to have a return type that is **more derived** than the type defined in the delegate. **Contravariance** permits a method to accept parameters that are **less derived** than the parameters of the delegate type. Variance support includes both **generic and non-generic delegates**. Custom delegates and strongly typed generic delegates (`Func`, `Action`) can use variance for greater flexibility. The effects of variance in delegate binding work with all delegate types, not just generic delegates with variant type parameters. Variance in delegate binding enables a method to be bound to any delegate that has more restrictive parameter types and a less restrictive return type.

**Beginner-Friendly Explanation**
Variance in delegates means **a delegate can point to a method with a slightly different signature** as long as the differences are safe. For covariance, a method returning `Dog` can be assigned to a delegate expecting `Animal`. For contravariance, a method accepting `Animal` can be assigned to a delegate expecting `Dog`.

---

### Purposes

- To allow **method group conversion** to delegates with compatible but not identical signatures.
- To enable **covariant return types** in delegates.
- To enable **contravariant parameter types** in delegates.
- To increase **flexibility** in event handling and callbacks.
- To support **custom delegate types** with variance semantics.

---

### Syntax Rules and Structure (Delegate Variance Rules)

| Scenario | Allowed? | Example |
|---|---|---|
| **Covariant return** | ✔️ | Method returns `Dog`, delegate expects `Animal` |
| **Contravariant parameter** | ✔️ | Method accepts `Animal`, delegate expects `Dog` |
| **Both covariance and contravariance** | ✔️ | Method returns `Dog`, accepts `Animal`; delegate expects `Animal`, passes `Dog` |
| **Value types** | ❌ | Variance does not apply to value types |
| **`ref` / `out` parameters** | ❌ | Ref-style parameters block variance |

**Component breakdown**

- **Covariant return** — More derived return type is allowed.
- **Contravariant parameter** — Less derived parameter type is allowed.
- **Custom delegates** — Variance applies to user-defined delegates.

#### Syntax Rules

- Variance in delegate binding works with **all delegate types** (generic and non-generic).
- Generic delegate variance requires the delegate type definition to have variant type parameters (`in`/`out`).
- Custom delegates can be covariant and contravariant in their signatures.

#### Constraints and Limitations

- Value types do not support variance.
- `ref`, `in`, and `out` parameters cannot be variant.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Variance with Custom Delegate

```csharp
using System;

public class Animal { }
public class Dog : Animal { }

public delegate Animal AnimalDelegate(Dog dog);

class Program
{
    public static Animal GetAnimal(Dog dog) => new Animal();          // exact match
    public static Dog GetDog(Dog dog) => new Dog();                    // covariant return
    public static Animal GetAnimalFromAnimal(Animal animal) => new Animal(); // contravariant parameter

    static void Main()
    {
        AnimalDelegate del;

        del = GetAnimal;              // exact match
        Animal a1 = del(new Dog());

        del = GetDog;                 // covariance: Dog return → Animal
        Animal a2 = del(new Dog());

        del = GetAnimalFromAnimal;    // contravariance: Animal param → Dog
        Animal a3 = del(new Dog());

        Console.WriteLine(a1 is Animal);   // True
        Console.WriteLine(a2 is Dog);      // True
        Console.WriteLine(a3 is Animal);   // True
    }
}
```

**Expected Output**

```
True
True
True
```

**Why this output occurs**

- `GetDog` is covariant in its return type; `GetAnimalFromAnimal` is contravariant in its parameter type. Both are assignable to `AnimalDelegate`.

---

#### Example 2: Covariance with `Func<TResult>`

```csharp
using System;

public class Person { }
public class Employee : Person { }

class Program
{
    public static Employee FindEmployee(string title) => new Employee();

    static void Main()
    {
        Func<string, Person> func = FindEmployee;   // covariance
        Person person = func("Manager");
        Console.WriteLine(person is Employee);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `Func<in T, out TResult>` is covariant in `TResult`; a method returning `Employee` can be assigned to `Func<string, Person>`.

---

#### Example 3: Combined Variance in Delegate Binding and Generic Type Parameters

```csharp
using System;

public class Type1 { }
public class Type2 : Type1 { }
public class Type3 : Type2 { }

class Program
{
    public static Type3 MyMethod(Type1 t) => t as Type3 ?? new Type3();

    static void Main()
    {
        Func<Type2, Type2> f1 = MyMethod;   // covariant return, contravariant param
        Func<Type3, Type1> f2 = f1;         // generic variance: Type2 → Type3 in, Type2 → Type1 out

        Type1 t1 = f2(new Type3());
        Console.WriteLine(t1 is Type3);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- The first assignment uses delegate binding variance; the second uses generic type parameter variance on `Func<T, TResult>`.

---

### Real-World Cases

- **LINQ** — `Func<T, TResult>` variance enables flexible query projections.
- **Event handlers** — Contravariant `Action<T>` for handling derived event args.
- **Callbacks** — Covariant `Func<TResult>` for factory delegates.

---

### References

- Variance in Delegates (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-delegates
- Examine variance in delegates — Training - https://learn.microsoft.com/en-us/training/modules/get-started-delegates/05-examine-variance-delegates
- Using Variance for Func and Action Generic Delegates (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/using-variance-for-func-and-action-generic-delegates

---

## 5. Variance Best Practices

### Definitions

**Core Definition**
Variance best practices are the guidelines for **correctly declaring and using** covariant and contravariant type parameters, ensuring type safety, maximizing flexibility, and avoiding common pitfalls such as invalid variance annotations.

**Technical Definition**
Best practices for variance derive from the C# language specification and .NET design guidelines. Key recommendations include: **use `out` for type parameters that appear only in return positions**; **use `in` for type parameters that appear only in parameter positions**; **do not use variance with value types** (it is not supported); **avoid `ref`, `in`, and `out` parameters in variant interfaces and delegates**; **prefer variant interfaces like `IEnumerable<out T>` and `IComparer<in T>`** where appropriate; **understand that variance is a compile-time feature** that does not change runtime behaviour; and **use variance to increase API flexibility** without compromising type safety. Variance should be **applied deliberately** and **documented** when designing generic APIs.

**Beginner-Friendly Explanation**
Best practices for variance are simple: **use `out` for outputs and `in` for inputs**, **do not try to use variance with value types or `ref` parameters**, and **prefer existing variant interfaces** like `IEnumerable<out T>`.

---

### Purposes

- To **correctly declare** variant interfaces and delegates.
- To **maximize flexibility** while preserving type safety.
- To **avoid compiler errors** from invalid variance annotations.
- To choose the right variance kind for the API design.
- To **document intent** clearly through `in`/`out` annotations.

---

### Guidelines and Structure

#### General Guidelines

```
// Use out for covariant (output-only) type parameters
interface IProducer<out T> { T Produce(); }

// Use in for contravariant (input-only) type parameters
interface IConsumer<in T> { void Consume(T item); }

// Use both for mixed variance (different type parameters)
interface ITransformer<out TResult, in TInput>
{
    TResult Transform(TInput input);
}

// Prefer existing variant interfaces (IEnumerable<out T>, IComparer<in T>)
// Do not use ref, in, or out parameters in variant interfaces
// Do not use variance with value types
```

**Component breakdown**

- **`out` for returns** — Covariant.
- **`in` for parameters** — Contravariant.
- **Both** — Different type parameters can have different variance.
- **Existing interfaces** — `IEnumerable<out T>`, `IComparer<in T>`.
- **No ref-style** — `ref`/`in`/`out` parameters block variance.
- **No value types** — Variance applies only to reference types.

#### Rules and Constraints

- **Do** use `out` for output-only type parameters.
- **Do** use `in` for input-only type parameters.
- **Do** use both `in` and `out` for different type parameters when needed.
- **Do** prefer built-in variant interfaces where possible.
- **Do not** use variance with value types.
- **Do not** use `ref`, `in`, or `out` parameters in variant interfaces or delegates.
- **Do not** use a covariant type parameter as a generic constraint.
- **Do not** use a contravariant type parameter as a return type.

---

### Annotated Code Examples and Expected Outputs

#### Example: Correctly Declared Variant Interface

```csharp
using System;
using System.Collections.Generic;

public interface IReadOnlyRepository<out T>
{
    IEnumerable<T> GetAll();
    T GetById(int id);
}

public class Animal { public string Name = "Animal"; }
public class Dog : Animal { public Dog() { Name = "Dog"; } }

class Program
{
    static void Main()
    {
        IReadOnlyRepository<Dog> dogRepo = null!;
        IReadOnlyRepository<Animal> animalRepo = dogRepo;   // covariance

        Console.WriteLine("Variance is valid");
    }
}
```

**Expected Output**

```
Variance is valid
```

**Why this output occurs**

- `IReadOnlyRepository<out T>` uses `T` only in return positions, satisfying covariance rules.

---

### Real-World Cases

- **LINQ** — `IEnumerable<out T>` is the canonical covariant interface.
- **Comparison** — `IComparer<in T>` is the canonical contravariant interface.
- **API design** — `Func<out TResult>` and `Action<in T>` for delegates.

---

### References

- Covariance and Contravariance (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/
- Creating Variant Generic Interfaces (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/creating-variant-generic-interfaces
- Variance in Delegates (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-delegates
- Using Variance for Func and Action Generic Delegates (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/using-variance-for-func-and-action-generic-delegates

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Covariance / contravariance | Introduced in **C# 4.0** and **.NET Framework 4**; stable. |
| Value types and variance | **Not supported** — variance applies only to reference types. |
| `ref` / `in` / `out` parameters | **Cannot be variant** in interfaces or delegates. |
| Covariant type as generic constraint | **Not allowed** for interface methods. |
| Contravariant type as return type | **Not allowed**. |
| Runtime behaviour | Variance is a **compile-time** feature; it does not change runtime behaviour. |
| Custom delegates | Variance applies to **user-defined delegates** as well as `Func`/`Action`. |

---

## References (Consolidated)

- Covariance and Contravariance (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/
- Creating Variant Generic Interfaces (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/creating-variant-generic-interfaces
- Variance in Generic Interfaces (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-generic-interfaces
- Variance in Delegates (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-delegates
- Using Variance for Func and Action Generic Delegates (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/using-variance-for-func-and-action-generic-delegates
- Examine variance in delegates — Training - https://learn.microsoft.com/en-us/training/modules/get-started-delegates/05-examine-variance-delegates

---