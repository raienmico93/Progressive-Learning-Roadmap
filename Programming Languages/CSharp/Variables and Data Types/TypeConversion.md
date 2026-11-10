# C# Type Conversion — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Type conversion in C# is the process of changing a value from one data type to another. C# is a statically typed language, meaning a variable, once declared, cannot be assigned a value of a different type unless an implicit or explicit conversion exists between those types. Conversions range from safe, automatic widening conversions (like `int` to `long`) to potentially lossy explicit casts (like `double` to `int`), and from parsing textual data into numbers to navigating reference-type hierarchies with runtime checks.

**Technical Definition**
A conversion is the process of changing a value from one type to another. A cast is the explicit syntax for conversion, written with parentheses like `(int)value`. An **implicit conversion** is a conversion that happens automatically when the compiler can guarantee it is safe—no syntax is required. An **explicit cast** is a conversion written in code, indicating the conversion might lose information or fail. C# supports several conversion categories: implicit numeric conversions (widening), explicit numeric conversions (narrowing), implicit reference conversions (derived to base, interface implementation, to `object`), explicit reference conversions (base to derived, requires runtime check), boxing/unboxing conversions, user-defined conversions, and parsing conversions via `Parse`/`TryParse`.

**Beginner-Friendly Explanation**
Type conversion is **changing the “shape” of your data** so it fits somewhere else. Imagine you have water in a small cup (an `int`) and need to pour it into a large bucket (a `long`)—that is easy and automatic (implicit). But pouring from a large bucket into a small cup might overflow—C# makes you explicitly acknowledge the risk with a cast `(int)`. Sometimes you have a word written on paper (a `string`) and need it as a number; that requires **parsing**, and the safest way is `TryParse`, which tells you if it worked instead of crashing.

---

### Key Characteristics

- **Two main directions** — Widening (safe, implicit) and narrowing (risky, explicit).
- **Compile-time safety** — Implicit conversions are proven safe by the compiler; explicit conversions are the programmer’s responsibility.
- **Reference vs. value semantics** — Reference conversions change the *view* of an object, not the object itself; numeric conversions change the *value*.
- **Runtime checks** — Explicit reference casts may throw `InvalidCastException`; the `is` and `as` operators provide safe alternatives.
- **Multiple tool families** — Cast expressions, `is`/`as` operators, `typeof`, `GetType`, the `Convert` class, and `Parse`/`TryParse` each serve distinct conversion scenarios.
- **Boxing is a conversion** — Converting a value type to `object` or an interface it implements is a special conversion with performance implications.

---

### Prerequisites

- Understanding of C#’s built-in types (`int`, `double`, `string`, `object`, etc.).
- Familiarity with value types vs. reference types and inheritance.
- Knowledge of interfaces and class hierarchies.
- Awareness of exceptions (`InvalidCastException`, `FormatException`).
- Basic understanding of compile-time vs. runtime behaviour.

---

### Related Programming Areas

- **Polymorphism** — Reference conversions are fundamental to working with base classes and interfaces.
- **Serialization and parsing** — Converting between text and typed data is central to JSON, XML, and configuration processing.
- **Interoperability** — Numeric and reference conversions are essential for P/Invoke and COM interop.
- **Reflection** — `typeof` and `GetType` are entry points to runtime type inspection.
- **Performance engineering** — Understanding boxing and conversion costs is key to writing efficient code.

---

### Core Concepts / Features

1. Implicit Conversion
2. Explicit Conversion
3. Casting
4. `as` Operator
5. `is` Operator
6. `is` Pattern
7. `typeof()` Operator
8. `GetType()`
9. `Convert` Class
10. `Parse()`
11. `TryParse()`
12. Type Conversion Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Implicit Conversion

### Definitions

**Core Definition**
An implicit conversion is a type conversion that the compiler performs automatically because it is guaranteed to succeed without losing information.

**Technical Definition**
An implicit conversion always succeeds and never loses data. No special syntax is required. Implicit numeric conversions occur when the destination type’s range and precision can fully accommodate the source type (widening conversions). Implicit reference conversions occur when assigning a derived class instance to a base class variable, assigning a reference type to an interface it implements, or assigning any reference type to `object`—these are safe because the derived type contains all members of the base type. The C# language specification defines a fixed set of implicit conversions; user-defined implicit conversions can also be declared.

**Beginner-Friendly Explanation**
Implicit conversion is **automatic and safe**. When you put a small value into a bigger container, C# does it for you without complaint. For example, an `int` fits perfectly into a `long`, so `long big = 42;` just works.

---

### Purposes

- To allow seamless assignment between compatible types without boilerplate.
- To support widening numeric conversions safely.
- To enable polymorphism by allowing derived types to be used where base types are expected.
- To simplify code by removing unnecessary casts.
- To ensure compile-time safety for conversions that cannot fail.

---

### Syntax Rules and Structure

Implicit conversions require no special syntax; they occur through assignment, parameter passing, or return statements:

```
// Implicit numeric conversion
int   small   =   42;
long   big   =   small   ;          // implicit widening

// Implicit reference conversion
Derived   d   =   new   Derived()   ;
Base   b   =   d   ;                 // derived to base
IInterface   i   =   d   ;           // to implemented interface
object   o   =   d   ;               // to object
```

**Component breakdown**

- `int` → `long` — Widening numeric conversion.
- `Derived` → `Base` — Implicit reference conversion (subtype to supertype).
- `Derived` → `IInterface` — Implicit conversion to implemented interface.
- `Derived` → `object` — Implicit conversion to `object`.

#### Syntax Rules

- Implicit numeric conversions exist for widening paths (e.g., `sbyte` → `short` → `int` → `long` → `float` → `double`).
- Implicit reference conversions always exist from a class to any base class or implemented interface.
- No data is lost; the compiler can prove safety.
- User-defined implicit conversions can be declared with `implicit operator`.

#### Constraints and Limitations

- Narrowing conversions (e.g., `long` to `int`) are **not** implicit; they require explicit casts.
- Implicit conversions do not exist between unrelated types (e.g., `string` to `int`).
- Implicit user-defined conversions should be used sparingly because they can surprise readers.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Widening Numeric Conversion

```csharp
using System;

class Program
{
    static void Main()
    {
        int itemCount = 42;
        long widened = itemCount;   // implicit conversion: int → long
        Console.WriteLine(widened); // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- Every `int` fits into a `long`, so the compiler performs the conversion automatically without any cast.

---

#### Example 2: Implicit Reference Conversion

```csharp
using System;

class Animal { public string Name { get; set; } }
class Mammal : Animal { }

class Program
{
    static void Main()
    {
        Mammal otter = new Mammal { Name = "River otter" };
        Animal animal = otter;          // derived to base: implicit
        object obj = otter;              // any type to object: implicit

        Console.WriteLine(animal.Name);  // River otter
    }
}
```

**Expected Output**

```
River otter
```

**Why this output occurs**

- A `Mammal` **is an** `Animal`, so assigning it to an `Animal` variable is always safe and implicit.

---

### Real-World Cases

- **Collections** — Storing `List<Mammal>` elements in `IEnumerable<Animal>` because `IEnumerable<out T>` is covariant.
- **Method parameters** — Passing an `int` to a method expecting a `double`.
- **Polymorphism** — Treating derived objects as their base type.

---

### References

- Type conversions, casting, and boxing — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/conversions
- Casting and type conversions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/casting-and-type-conversions

---

## 2. Explicit Conversion

### Definitions

**Core Definition**
An explicit conversion is a type conversion that the programmer must explicitly request, typically because the conversion might lose information or fail at runtime.

**Technical Definition**
Explicit conversions require a **cast expression**—the destination type in parentheses before the value. Casting is required when information might be lost in the conversion, or when the conversion might not succeed for other reasons. Typical examples include numeric conversion to a type with less precision or a smaller range, and conversion of a base-class instance to a derived class. For reference types, an explicit cast does not change the runtime type of the object; it changes the type of the reference used to view the object. If the cast fails at runtime, an `InvalidCastException` is thrown.

**Beginner-Friendly Explanation**
Explicit conversion is **“I know what I’m doing—force it.”** When you pour a large bucket into a small cup, you might spill. C# makes you write `(int)` to say “yes, I accept the risk that data might be lost or the conversion might fail.”

---

### Purposes

- To perform narrowing numeric conversions where precision or range may be lost.
- To convert a base-type reference back to a derived type (downcasting).
- To implement user-defined conversions between custom types.
- To convert between incompatible types via helper classes.
- To take explicit responsibility for potentially unsafe conversions.

---

### Syntax Rules and Structure

```
destinationType   variable   =   (destinationType)expression   ;
```

**Component breakdown**

- `destinationType` — The target type.
- `(destinationType)` — The cast expression.
- `expression` — The value to convert.

#### Syntax Rules

- Numeric narrowing (e.g., `double` to `int`) truncates fractional parts.
- Reference downcasts compile but may throw `InvalidCastException` at runtime.
- User-defined explicit conversions use `explicit operator`.
- The cast operator has high precedence.

#### Constraints and Limitations

- Casting does not convert the underlying object for reference types; it only changes the reference type used to view it.
- Overflow in unchecked contexts wraps silently; use `checked` to detect overflow.
- `InvalidCastException` is thrown when a reference cast is invalid at runtime.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Numeric Narrowing

```csharp
using System;

class Program
{
    static void Main()
    {
        double average = 19.75;
        int truncated = (int)average;   // explicit cast: truncates
        Console.WriteLine(truncated);  // 19
    }
}
```

**Expected Output**

```
19
```

**Why this output occurs**

- The explicit cast `(int)` truncates the fractional part. The compiler requires the cast because `double` → `int` is a narrowing conversion.

---

#### Example 2: Reference Downcast and Runtime Failure

```csharp
using System;

class Animal { }
class Mammal : Animal { }
class Reptile : Animal { }

class Program
{
    static void Main()
    {
        Animal a = new Mammal();
        // Reptile r = (Reptile)a;   // InvalidCastException at runtime

        Animal safe = new Mammal();
        if (safe is Mammal m)
            Console.WriteLine("Successfully downcast to Mammal");
    }
}
```

**Expected Output**

```
Successfully downcast to Mammal
```

**Why this output occurs**

- The commented line would compile but throw `InvalidCastException` at runtime because the object is not a `Reptile`.
- The `is` pattern checks the runtime type before casting, making the downcast safe.

---

### Real-World Cases

- **Deserialization** — Casting deserialized objects back to their concrete types.
- **Interop** — Converting between numeric types of different sizes for native APIs.
- **Mathematical algorithms** — Truncating floating-point results to integers.

---

### References

- Casting and type conversions — C# Programming Guide — Explicit conversions - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/casting-and-type-conversions
- Type conversions, casting, and boxing — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/conversions

---

## 3. Casting

### Definitions

**Core Definition**
Casting is the act of performing an explicit conversion using the cast expression syntax `(T)E`.

**Technical Definition**
A cast expression of the form `(T)E` explicitly converts the result of expression `E` to type `T`. If no explicit conversion exists from the type of `E` to type `T`, the compiler issues an error. At runtime, an explicit conversion might not succeed, and a cast expression might throw an exception. Casting is the mechanism that implements explicit conversions, including numeric narrowing and reference downcasting. It does not change the runtime type of an object; it changes the type of the reference through which the object is viewed.

**Beginner-Friendly Explanation**
Casting is **putting parentheses around a type** to force a conversion. `(int)someDouble` means “treat this `double` as an `int`.” It works for many conversions, but if the conversion is impossible, the program will crash.

---

### Purposes

- To perform explicit numeric conversions.
- To downcast reference types to more specific types.
- To invoke user-defined conversion operators.
- To convert between compatible numeric types when the compiler cannot prove safety.
- To access members of a derived type through a base-typed reference.

---

### Syntax Rules and Structure

```
(T)E
```

**Component breakdown**

- `T` — The target type.
- `E` — The expression to convert.
- `(T)` — The cast expression.

#### Syntax Rules

- The cast applies to the immediately following expression.
- If no explicit conversion exists, a compile-time error occurs.
- At runtime, an invalid reference cast throws `InvalidCastException`.
- Numeric casts in unchecked contexts may silently overflow.

#### Constraints and Limitations

- Cannot be used for conversions that have no explicit definition.
- Reference casts do not change the underlying object.
- Overflow behaviour depends on `checked`/`unchecked` context.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Numeric Cast

```csharp
using System;

class Program
{
    static void Main()
    {
        double x = 1234.7;
        int a = (int)x;               // cast expression
        Console.WriteLine(a);         // 1234
    }
}
```

**Expected Output**

```
1234
```

**Why this output occurs**

- The cast truncates the fractional part. Without the cast, the compiler would reject the assignment.

---

#### Example 2: Casting with `checked` Context

```csharp
using System;

class Program
{
    static void Main()
    {
        int big = int.MaxValue;
        try
        {
            int overflow = checked((int)(big + 1L));
        }
        catch (OverflowException)
        {
            Console.WriteLine("Overflow detected");
        }
    }
}
```

**Expected Output**

```
Overflow detected
```

**Why this output occurs**

- The `checked` context causes an `OverflowException` when the `long` result exceeds the `int` range, rather than silently wrapping.

---

### Real-World Cases

- **Parsing binary formats** — Casting byte sequences to numeric types.
- **UI frameworks** — Casting `object` event senders to specific control types.
- **Mathematical computations** — Converting `double` to `int` for indexing.

---

### References

- Cast expression — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#cast-expression
- Casting and type conversions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/casting-and-type-conversions

---

## 4. `as` Operator

### Definitions

**Core Definition**
The `as` operator attempts a reference or nullable value type conversion and returns `null` if the conversion fails, instead of throwing an exception.

**Technical Definition**
The `as` operator explicitly converts an expression to a given reference or nullable value type. If the conversion is not possible, `as` returns `null`. Unlike a cast expression, the `as` operator never throws an exception. The expression `E as T` is equivalent to `E is T ? (T)E : (T)null`, except that `E` is evaluated only once. The `as` operator considers only reference, nullable, boxing, and unboxing conversions; it cannot be used for user-defined conversions or numeric conversions.

**Beginner-Friendly Explanation**
The `as` operator is a **safe cast**. It tries to convert, and if it cannot, it gives you `null` instead of crashing. You must check for `null` afterward. It is like asking “Can you be this type? If not, just tell me ‘no’ quietly.”

---

### Purposes

- To safely attempt a reference-type conversion without exceptions.
- To simplify code that would otherwise use `is` followed by a cast.
- To obtain a nullable result when a conversion might fail.
- To work with base-typed references that may or may not be a specific derived type.

---

### Syntax Rules and Structure

```
expression   as   T
```

**Component breakdown**

- `expression` — The value to convert.
- `as` — The operator.
- `T` — A reference type or nullable value type.

#### Syntax Rules

- Returns `null` if the conversion fails.
- Only reference, nullable, boxing, and unboxing conversions are considered.
- Cannot be used for user-defined conversions or numeric conversions.
- `T` must be a reference type or nullable value type.

#### Constraints and Limitations

- Cannot be used with non-nullable value types (use `(T)` with `is` instead).
- The result must be checked for `null` before use.
- Does not consider user-defined conversion operators.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Interface Conversion

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        IEnumerable<int> numbers = new List<int> { 10, 20, 30 };
        IList<int> indexable = numbers as IList<int>;

        if (indexable != null)
        {
            Console.WriteLine(indexable[0] + indexable[indexable.Count - 1]); // 40
        }
    }
}
```

**Expected Output**

```
40
```

**Why this output occurs**

- `List<int>` implements `IList<int>`, so the `as` conversion succeeds. The result is checked for `null` before use.

---

#### Example 2: Failed Conversion Returns Null

```csharp
using System;

class Animal { }
class Mammal : Animal { }
class Reptile : Animal { }

class Program
{
    static void Main()
    {
        object boxedReptile = new Reptile();
        Mammal? maybeMammal = boxedReptile as Mammal;

        Console.WriteLine(maybeMammal is null ? "Safe null result" : "Unexpected");
    }
}
```

**Expected Output**

```
Safe null result
```

**Why this output occurs**

- A `Reptile` cannot be converted to `Mammal`, so `as` returns `null` without throwing.

---

### Real-World Cases

- **Event handlers** — Casting `sender` to a specific control type safely.
- **Plugin systems** — Checking whether an object implements a plugin interface.
- **Deserialization** — Attempting to cast deserialized objects to expected types.

---

### References

- The as operator — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-as-operator
- Type conversions, casting, and boxing — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/conversions

---

## 5. `is` Operator

### Definitions

**Core Definition**
The `is` operator checks whether the runtime type of an expression is compatible with a given type, returning a Boolean result.

**Technical Definition**
The `is` operator checks if the runtime type of an expression result is compatible with a given type. It returns `true` when the expression result is non-null and its runtime type has an identity conversion to `T`, derives from `T`, implements interface `T`, has an implicit reference conversion to `T`, is a nullable value type with underlying type `T` and `HasValue` is `true`, or a boxing/unboxing conversion exists. The `is` operator does **not** consider user-defined conversions or implicit span conversions. It is purely a runtime type check.

**Beginner-Friendly Explanation**
The `is` operator is a **“type question.”** `obj is Mammal` asks: “At runtime, is this object a `Mammal` (or something derived from it)?” It returns `true` or `false` without attempting any conversion.

---

### Purposes

- To test whether an object is compatible with a specific type before casting.
- To avoid `InvalidCastException` by checking type compatibility first.
- To implement type-based branching logic.
- To validate inputs in APIs that accept `object`.
- To support polymorphic behaviour based on runtime type.

---

### Syntax Rules and Structure

```
expression   is   T
```

**Component breakdown**

- `expression` — The value to test.
- `is` — The type-testing operator.
- `T` — The type or pattern to test against.

#### Syntax Rules

- Returns `true` if the expression’s runtime type is compatible with `T`.
- Returns `false` for `null`.
- Considers identity, reference, boxing, and unboxing conversions.
- Does not consider numeric conversions or user-defined conversions.

#### Constraints and Limitations

- Does not perform any conversion; only tests.
- For value types, `is` checks the exact type (boxing) or nullable `HasValue`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Type Test

```csharp
using System;

class Animal { }
class Mammal : Animal { }

class Program
{
    static void Main()
    {
        object a = new Mammal();
        Console.WriteLine(a is Animal);   // True
        Console.WriteLine(a is Mammal);   // True
        Console.WriteLine(a is string);   // False
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

- A `Mammal` is also an `Animal` due to inheritance.
- The object is not a `string`.

---

#### Example 2: `is` with Boxing and Nullable

```csharp
using System;

class Program
{
    static void Main()
    {
        int i = 27;
        object boxed = i;
        Console.WriteLine(boxed is int);    // True
        Console.WriteLine(boxed is long);   // False (no numeric conversion)

        int? nullable = 7;
        Console.WriteLine(nullable is int); // True (HasValue is true)
    }
}
```

**Expected Output**

```
True
False
True
```

**Why this output occurs**

- Boxing preserves the `int` type.
- `is` does not consider numeric conversions (`int` → `long`).
- A nullable with a value matches its underlying type.

---

### Real-World Cases

- **Event handling** — Checking whether `sender` is a specific control type.
- **Plugin validation** — Verifying that an object implements a required interface.
- **Visitor patterns** — Dispatching behaviour based on runtime type.

---

### References

- The is operator — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-is-operator
- Type conversions, casting, and boxing — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/conversions

---

## 6. `is` Pattern (Declaration Pattern)

### Definitions

**Core Definition**
The `is` pattern (declaration pattern) tests an expression against a type and, if successful, **assigns the converted value to a new variable** in a single expression.

**Technical Definition**
The `is` pattern extends the `is` operator with **pattern matching**. In the form `expression is T variable`, if the runtime type of `expression` is compatible with `T`, the pattern succeeds and `variable` is assigned the converted value; otherwise, the pattern fails. The variable is definitely assigned only when the pattern succeeds, and its scope is limited to the enclosing block where the pattern is true. This is the recommended alternative to `as` followed by a `null` check because it keeps the test and assignment together.

**Beginner-Friendly Explanation**
The `is` pattern is **“check and grab in one step.”** Instead of checking if something is a `Mammal` and then casting it, you write `if (animal is Mammal m)`—now `m` is a `Mammal` variable you can use directly inside the `if` block.

---

### Purposes

- To combine type checking and conversion in a single expression.
- To avoid duplicate casts and improve readability.
- To keep the converted variable scoped to where it is valid.
- To enable safe downcasting without `as` and `null` checks.
- To support pattern-based dispatch.

---

### Syntax Rules and Structure

```
expression   is   T   variable
```

**Component breakdown**

- `expression` — The value to test.
- `is` — The pattern-matching operator.
- `T` — The target type.
- `variable` — The variable assigned when the pattern succeeds.

#### Syntax Rules

- The variable is in scope only where the pattern is true.
- The variable is definitely assigned inside the `true` branch.
- The pattern fails for `null`.
- Can be combined with `&&` for additional conditions.

#### Constraints and Limitations

- The variable cannot be used outside the successful branch.
- Pattern variables cannot shadow existing variables in the same scope in ways that cause ambiguity.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Downcast with `is` Pattern

```csharp
using System;

class Animal { public string Name { get; set; } }
class Mammal : Animal { }

class Program
{
    static void Main()
    {
        Animal knownAnimal = new Mammal { Name = "River otter" };

        if (knownAnimal is Mammal mammal)
        {
            Console.WriteLine($"Pattern match succeeded: {mammal.Name}");
        }
    }
}
```

**Expected Output**

```
Pattern match succeeded: River otter
```

**Why this output occurs**

- The `is` pattern tests and casts in one step, assigning `mammal` only when the runtime type is compatible.

---

#### Example 2: Pattern with Additional Condition

```csharp
using System;

class Program
{
    static void Main()
    {
        object value = 42;

        if (value is int number && number > 10)
        {
            Console.WriteLine($"Large number: {number}");
        }
    }
}
```

**Expected Output**

```
Large number: 42
```

**Why this output occurs**

- The `is int number` pattern succeeds and assigns `42` to `number`; the `&&` condition then checks the value.

---

### Real-World Cases

- **Visitor patterns** — Type-based dispatch with extracted values.
- **API input validation** — Checking and extracting typed parameters.
- **Game engines** — Collision handling based on object types.

---

### References

- Type testing with pattern matching — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#type-testing-with-pattern-matching
- Type conversions, casting, and boxing — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/conversions

---

## 7. `typeof()` Operator

### Definitions

**Core Definition**
The `typeof` operator obtains the `System.Type` instance for a type known at compile time.

**Technical Definition**
The `typeof` operator returns the `System.Type` object for a type or type parameter specified as its argument. The argument must be the name of a type or a type parameter, not an expression. The argument cannot be a type that requires metadata annotations, such as `dynamic` or nullable reference types (`string?`); use the underlying type instead (`object`, `string`). `typeof` can also be used with unbound generic types (e.g., `typeof(Dictionary<,>)`). To get the `Type` of an expression’s **runtime** type, use `Object.GetType()` instead.

**Beginner-Friendly Explanation**
`typeof(SomeType)` is **“give me the type object for `SomeType`.”** It is a compile-time way to get information about a type, such as its name or whether it is an interface. You use it when you know the type name at compile time.

---

### Purposes

- To obtain a `System.Type` instance for reflection.
- To pass type information to APIs that accept `Type` parameters.
- To compare types at runtime (`GetType() == typeof(T)`).
- To create instances via reflection (`Activator.CreateInstance(typeof(T))`).
- To inspect metadata such as methods, properties, and attributes.

---

### Syntax Rules and Structure

```
typeof(T)
```

**Component breakdown**

- `typeof` — The operator.
- `T` — The type name or type parameter.

#### Syntax Rules

- The argument must be a type name, not an expression.
- Cannot be used with `dynamic` or nullable reference types; use the underlying type.
- Unbound generic types use commas for type parameter count (`Dictionary<,>`).

#### Constraints and Limitations

- Does not accept variables or runtime expressions.
- Cannot represent nullable reference type annotations.
- Returns a `System.Type` object, not an instance of the type.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `typeof` Usage

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine(typeof(int));              // System.Int32
        Console.WriteLine(typeof(List<string>));     // System.Collections.Generic.List`1[System.String]
        Console.WriteLine(typeof(Dictionary<,>));    // System.Collections.Generic.Dictionary`2[TKey,TValue]
    }
}
```

**Expected Output**

```
System.Int32
System.Collections.Generic.List`1[System.String]
System.Collections.Generic.Dictionary`2[TKey,TValue]
```

**Why this output occurs**

- `typeof` returns the `Type` object for each type, including constructed and unbound generic types.

---

#### Example 2: `typeof` vs. `GetType`

```csharp
using System;

class Animal { }
class Giraffe : Animal { }

class Program
{
    static void Main()
    {
        object b = new Giraffe();
        Console.WriteLine(b is Animal);                   // True
        Console.WriteLine(b.GetType() == typeof(Animal)); // False (runtime type is Giraffe)
        Console.WriteLine(b.GetType() == typeof(Giraffe)); // True
    }
}
```

**Expected Output**

```
True
False
True
```

**Why this output occurs**

- `is` checks compatibility (a `Giraffe` is an `Animal`).
- `GetType()` returns the exact runtime type (`Giraffe`), which does not equal `typeof(Animal)`.

---

### Real-World Cases

- **Dependency injection** — Registering services by `Type`.
- **Serialization** — Inspecting type metadata for JSON/XML serialization.
- **Dynamic instantiation** — `Activator.CreateInstance(typeof(T))`.

---

### References

- The typeof operator — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-typeof-operator
- Object.GetType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.gettype

---

## 8. `GetType()`

### Definitions

**Core Definition**
`GetType()` is an instance method inherited from `System.Object` that returns the exact runtime type of the current instance.

**Technical Definition**
`Object.GetType()` returns a `System.Type` instance representing the **runtime type** of the object. Because `System.Object` is the base class for all types in .NET, `GetType()` is available on every object, including boxed value types. For two objects `x` and `y` that have identical runtime types, `Object.ReferenceEquals(x.GetType(), y.GetType())` returns `true`. Unlike `typeof`, which is resolved at compile time from a type name, `GetType()` is resolved at runtime from an instance. Do not override `GetType()`; it is non-virtual.

**Beginner-Friendly Explanation**
`GetType()` is **“what am I, really?”** Every object can tell you its exact type at runtime. Unlike `typeof`, which asks about a type name you write in code, `GetType()` asks an actual object what its runtime type is.

---

### Purposes

- To discover the exact runtime type of an object.
- To implement type-based logic when the compile-time type is `object` or a base type.
- To compare runtime types for equality.
- To drive reflection-based operations on instances.
- To support diagnostic and logging scenarios.

---

### Syntax Rules and Structure

```
objectInstance.GetType()
```

**Component breakdown**

- `objectInstance` — Any object (including boxed value types).
- `GetType()` — The instance method.

#### Syntax Rules

- Returns a `System.Type` object.
- Returns the **exact** runtime type, not a base type.
- Available on all types because it is inherited from `System.Object`.
- Cannot be overridden.

#### Constraints and Limitations

- Not the same as `typeof`; `typeof` uses a compile-time type name.
- For nullable value types, boxing produces the underlying type’s `Type`.
- Reflection-based and therefore has performance overhead compared to `typeof`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Runtime Type Discovery

```csharp
using System;

class Animal { }
class Dog : Animal { }

class Program
{
    static void Main()
    {
        Animal a = new Dog();
        Console.WriteLine(a.GetType());              // Dog
        Console.WriteLine(a.GetType() == typeof(Dog)); // True
    }
}
```

**Expected Output**

```
Dog
True
```

**Why this output occurs**

- `GetType()` returns the runtime type `Dog`, not the compile-time type `Animal`.

---

#### Example 2: `GetType` on Boxed Value Type

```csharp
using System;

class Program
{
    static void Main()
    {
        object boxed = 42;
        Console.WriteLine(boxed.GetType());   // System.Int32
    }
}
```

**Expected Output**

```
System.Int32
```

**Why this output occurs**

- Boxing preserves the runtime type, so `GetType()` returns `System.Int32`.

---

### Real-World Cases

- **Logging** — Recording the actual type of an object for diagnostics.
- **Serialization** — Determining the concrete type to serialize.
- **Equality checks** — Comparing runtime types in custom `Equals` implementations.

---

### References

- Object.GetType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.gettype
- The typeof operator — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-typeof-operator

---

## 9. `Convert` Class

### Definitions

**Core Definition**
The `Convert` class is a static utility class in `System` that provides methods for converting between the .NET base data types.

**Technical Definition**
`System.Convert` contains static methods primarily used to support conversion to and from the base data types in .NET: `Boolean`, `Char`, `SByte`, `Byte`, `Int16`, `Int32`, `Int64`, `UInt16`, `UInt32`, `UInt64`, `Single`, `Double`, `Decimal`, `DateTime`, and `String`. The most commonly used methods are `ToInt32`, `ToString`, `ToDouble`, and `ChangeType`. `Convert.ChangeType` converts an object to a specified type using the `IConvertible` interface. `Convert` methods throw `InvalidCastException`, `FormatException`, or `OverflowException` when conversion is not possible.

**Beginner-Friendly Explanation**
The `Convert` class is a **universal translator** for basic types. You can use it to turn a string into a number, a number into a string, or a boolean into an integer. It handles `null` gracefully (returning `0` or empty strings) and provides culture-aware conversions.

---

### Purposes

- To convert between base data types when no implicit or explicit cast exists.
- To handle `null` values gracefully (e.g., `Convert.ToInt32(null)` returns `0`).
- To perform culture-aware conversions.
- To convert an object to a specified type via `ChangeType`.
- To provide a consistent API for common conversions.

---

### Syntax Rules and Structure

```
Convert.ToTargetType(value)
Convert.ChangeType(value, targetType)
```

**Component breakdown**

- `ToTargetType` — Specific conversion method (e.g., `ToInt32`).
- `value` — The source value.
- `ChangeType` — Converts to a runtime-specified type.

#### Syntax Rules

- `Convert` methods are static.
- `null` is treated as `0` for numeric conversions and `""` for string conversions.
- `ChangeType` requires the target type to implement `IConvertible`.
- Culture can be specified via an `IFormatProvider` overload.

#### Constraints and Limitations

- Cannot convert between arbitrary types (use `TypeConverter` or custom logic).
- Throws exceptions for invalid formats or overflows.
- Less efficient than direct casts for numeric conversions.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: String to Number

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "123";
        int number = Convert.ToInt32(text);
        Console.WriteLine(number);   // 123

        string nullText = null;
        Console.WriteLine(Convert.ToInt32(nullText)); // 0
    }
}
```

**Expected Output**

```
123
0
```

**Why this output occurs**

- `Convert.ToInt32` parses the string; `null` is treated as `0`.

---

#### Example 2: `ChangeType` for Runtime Conversion

```csharp
using System;

class Program
{
    static void Main()
    {
        object value = "456";
        Type target = typeof(int);
        int result = (int)Convert.ChangeType(value, target);
        Console.WriteLine(result);   // 456
    }
}
```

**Expected Output**

```
456
```

**Why this output occurs**

- `ChangeType` converts the string to the specified `int` type at runtime.

---

### Real-World Cases

- **Configuration parsing** — Converting string settings to typed values.
- **Data binding** — Converting user input to model property types.
- **Interop** — Converting between COM and .NET types.

---

### References

- Convert Class (System) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.convert
- How to convert a string to a number — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/how-to-convert-a-string-to-a-number

---

## 10. `Parse()`

### Definitions

**Core Definition**
`Parse()` is a static method on numeric and other types that converts the string representation of a value into that type.

**Technical Definition**
`Parse` methods are available on numeric types (`int`, `long`, `double`, `decimal`, etc.), `DateTime`, `TimeSpan`, and other types. They convert a string to the target type and **throw an exception** (`FormatException` or `OverflowException`) if the string is not a valid representation. Parsing APIs create a new value from the source string; they do not modify the source. `Parse` should be used only when the input is guaranteed to be valid, such as controlled test data.

**Beginner-Friendly Explanation**
`Parse` is **“turn this text into a number, and crash if it’s not valid.”** It is fast and direct, but if the text is not a proper number, your program throws an exception.

---

### Purposes

- To convert text to numeric types when validity is assured.
- To parse dates, times, and other structured values.
- To provide fast conversion in controlled scenarios.
- To support culture-specific parsing via `IFormatProvider` overloads.

---

### Syntax Rules and Structure

```
TargetType   variable   =   TargetType.Parse(stringValue)   ;
```

**Component breakdown**

- `TargetType.Parse` — Static parse method.
- `stringValue` — The text to parse.

#### Syntax Rules

- Throws `FormatException` if the string is not in the correct format.
- Throws `OverflowException` if the value exceeds the target type’s range.
- Ignores leading and trailing whitespace.
- Overloads accept `IFormatProvider` and `NumberStyles`.

#### Constraints and Limitations

- Not suitable for untrusted user input; prefer `TryParse`.
- Exceptions are costly if input is frequently invalid.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Successful Parse

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "512";
        int parsed = int.Parse(text);
        Console.WriteLine(parsed);   // 512
    }
}
```

**Expected Output**

```
512
```

**Why this output occurs**

- `int.Parse` converts the numeric string to an `int`.

---

#### Example 2: Parse Failure

```csharp
using System;

class Program
{
    static void Main()
    {
        try
        {
            int parsed = int.Parse("12x");
        }
        catch (FormatException)
        {
            Console.WriteLine("Invalid format");
        }
    }
}
```

**Expected Output**

```
Invalid format
```

**Why this output occurs**

- The string `"12x"` is not a valid integer, so `Parse` throws `FormatException`.

---

### Real-World Cases

- **Test data** — Parsing known-valid constants.
- **Configuration** — Parsing values that have been pre-validated.
- **Protocols** — Parsing numeric fields from trusted sources.

---

### References

- How to convert a string to a number — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/how-to-convert-a-string-to-a-number
- Parse and TryParse — .NET numeric parsing - https://learn.microsoft.com/en-us/dotnet/standard/base-types/parsing-numeric

---

## 11. `TryParse()`

### Definitions

**Core Definition**
`TryParse()` is a static method on numeric and other types that attempts to convert a string to the target type and returns a Boolean indicating success, without throwing an exception on failure.

**Technical Definition**
`TryParse` methods are available on all primitive numeric types, `DateTime`, `IPAddress`, and other types. They return `true` if parsing succeeds and assign the result to an `out` parameter; otherwise they return `false` and set the `out` parameter to the type’s default value. `TryParse` is the recommended approach for user input, network payloads, and file data because it makes failure handling explicit and avoids the cost of exceptions.

**Beginner-Friendly Explanation**
`TryParse` is **“try to turn this text into a number; tell me if it worked instead of crashing.”** It is the safe way to parse user input.

---

### Purposes

- To safely parse user input without exceptions.
- To validate text before using it as a typed value.
- To make failure handling explicit through a Boolean result.
- To avoid the performance cost of exceptions in parsing loops.
- To support culture-specific parsing.

---

### Syntax Rules and Structure

```
bool   success   =   TargetType.TryParse(stringValue,   out   TargetType result)   ;
```

**Component breakdown**

- `TargetType.TryParse` — Static method.
- `stringValue` — The text to parse.
- `out result` — Receives the parsed value on success; default on failure.

#### Syntax Rules

- Returns `true` on success, `false` on failure.
- Does not throw exceptions for invalid format or overflow.
- Ignores leading and trailing whitespace.
- Overloads accept `IFormatProvider` and `NumberStyles`.

#### Constraints and Limitations

- The `out` variable is always assigned (default on failure).
- Does not distinguish between format errors and overflow errors.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Successful Parse

```csharp
using System;

class Program
{
    static void Main()
    {
        bool ok = int.TryParse("512", out int value);
        Console.WriteLine(ok);      // True
        Console.WriteLine(value);  // 512
    }
}
```

**Expected Output**

```
True
512
```

**Why this output occurs**

- `TryParse` succeeds and assigns `512` to `value`.

---

#### Example 2: Failed Parse with Fallback

```csharp
using System;

class Program
{
    static void Main()
    {
        string userInput = "12x";
        if (int.TryParse(userInput, out int safeValue))
        {
            Console.WriteLine($"Parsed: {safeValue}");
        }
        else
        {
            Console.WriteLine("Input is not a valid number."); // this branch
        }
    }
}
```

**Expected Output**

```
Input is not a valid number.
```

**Why this output occurs**

- `TryParse` returns `false` for invalid input and does not throw.

---

### Real-World Cases

- **User input validation** — Parsing form fields safely.
- **Configuration parsing** — Handling potentially malformed settings.
- **Network protocols** — Parsing untrusted payloads without exceptions.

---

### References

- How to convert a string to a number — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/how-to-convert-a-string-to-a-number
- Parse and TryParse — .NET numeric parsing - https://learn.microsoft.com/en-us/dotnet/standard/base-types/parsing-numeric

---

## 12. Type Conversion Best Practices

### Definitions

**Core Definition**
Type conversion best practices are the guidelines for choosing the right conversion mechanism, avoiding data loss, and handling failures safely.

**Technical Definition**
Best practices encompass preferring implicit conversions when safe, using explicit casts only with clear justification, choosing `is` patterns over `as`-plus-null checks, using `TryParse` for user-facing text, avoiding unnecessary boxing, and understanding the runtime behaviour of reference conversions. These guidelines are documented in Microsoft’s C# programming guide and .NET fundamentals documentation.

**Beginner-Friendly Explanation**
Best practices are the habits that keep conversion code safe and predictable: let the compiler handle safe conversions, be explicit when risk exists, use `TryParse` for user input, and prefer `is` patterns over dangerous casts.

---

### Purposes

- To prevent data loss and runtime exceptions.
- To write clear, self-documenting conversion code.
- To avoid performance pitfalls such as unnecessary boxing.
- To handle user input safely.
- To choose the most appropriate conversion API for each scenario.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer implicit conversions when safe
long big = 42;                     // int → long implicit

// Use explicit casts only when justified
int small = (int)someDouble;      // narrowing: acknowledge risk

// Prefer is patterns over as + null checks
if (obj is Mammal m) { ... }

// Use TryParse for user input
if (int.TryParse(input, out int n)) { ... }

// Use Parse only for guaranteed-valid input
int constant = int.Parse("42");

// Avoid unnecessary boxing
// Prefer generic collections over non-generic
```

**Component breakdown**

- **Implicit when safe** — No data loss, no syntax.
- **Explicit when risky** — Acknowledge potential loss.
- **`is` pattern** — Combines test and assignment, scoped correctly.
- **`TryParse`** — Safe parsing without exceptions.
- **`Parse`** — Only for guaranteed-valid data.

#### Rules and Constraints

- Do not use `as` when a null value could be a valid result; prefer `is` pattern.
- Always check `as` results for `null`.
- Use `TryParse` for all user input; reserve `Parse` for controlled data.
- Be explicit about culture in parsing and formatting.
- Avoid casting between unrelated types; use `Convert` or parsing instead.
- Understand that reference casts do not change the object’s runtime type.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Reference Conversion with `is` Pattern

```csharp
using System;

class Animal { }
class Mammal : Animal { public string Name { get; set; } }

class Program
{
    static void Main()
    {
        Animal animal = new Mammal { Name = "Otter" };

        if (animal is Mammal m)
        {
            Console.WriteLine(m.Name);   // Otter
        }
    }
}
```

**Expected Output**

```
Otter
```

**Why this output occurs**

- The `is` pattern tests and assigns in one step, avoiding a separate `as` and `null` check.

---

#### Example 2: Safe Parsing with `TryParse`

```csharp
using System;

class Program
{
    static void Main()
    {
        string input = "abc";
        if (int.TryParse(input, out int value))
        {
            Console.WriteLine(value);
        }
        else
        {
            Console.WriteLine("Invalid number"); // this branch
        }
    }
}
```

**Expected Output**

```
Invalid number
```

**Why this output occurs**

- `TryParse` returns `false` without throwing, making failure explicit.

---

### Real-World Cases

- **API design** — Accepting `object` and using `is` patterns to extract typed values.
- **Configuration** — Using `TryParse` for optional settings.
- **Data import** — Parsing CSV or user-uploaded files with `TryParse`.

---

### References

- Type conversions, casting, and boxing — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/conversions
- Casting and type conversions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/casting-and-type-conversions
- How to safely cast using pattern matching and the as and is operators - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/safely-cast-using-pattern-matching-is-and-as-operators

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `as` with non-nullable value types | **Not allowed**; use `is` pattern with a cast instead. |
| `Parse` for user input | **Risky**; throws exceptions on invalid input. Prefer `TryParse`. |
| Unchecked numeric overflow | **Unsafe**; silent wrapping. Use `checked` to detect overflow. |
| Reference casts and runtime type | Casting does **not** change the object’s runtime type; it only changes the reference view. |
| `Convert.ChangeType` | Requires target type to implement `IConvertible`; may not work with custom types. |
| `typeof` with nullable reference types | **Not allowed**; use the underlying type (`string` instead of `string?`). |
| `GetType()` performance | Reflection-based; slower than `typeof`. Use `typeof` for compile-time type comparisons. |
| Implicit user-defined conversions | Can cause **surprising behaviour**; use sparingly and document clearly. |

---

## References (Consolidated)

- Type conversions, casting, and boxing — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/conversions
- Casting and type conversions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/casting-and-type-conversions
- Type-testing operators and cast expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast
- The as operator — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-as-operator
- The is operator — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-is-operator
- The typeof operator — Type-testing operators and cast expressions - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-typeof-operator
- Object.GetType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.gettype
- Convert Class (System) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.convert
- How to convert a string to a number — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/how-to-convert-a-string-to-a-number
- Parse and TryParse — .NET numeric parsing - https://learn.microsoft.com/en-us/dotnet/standard/base-types/parsing-numeric
- How to safely cast using pattern matching and the as and is operators - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/safely-cast-using-pattern-matching-is-and-as-operators
- User-defined conversion operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/user-defined-conversion-operators
- Built-in numeric conversions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/numeric-conversions

---