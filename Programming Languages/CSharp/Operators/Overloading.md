# C# Operator Overloading — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Operator overloading is a C# language feature that allows user-defined types (classes and structs) to **redefine the behaviour** of predefined operators—such as `+`, `-`, `*`, `==`, `!=`, `<`, `>`, and conversion casts—when one or both operands are of the defining type. Instead of requiring callers to invoke named methods like `Add(a, b)` or `Equals(other)`, operator overloading lets custom types participate in C#’s native expression syntax: `a + b`, `a == b`, `(int)a`. This makes domain types such as `Complex`, `Money`, `Vector`, and `DateTime` feel like **primitive built-in types** to the code that uses them.

**Technical Definition**
All unary and binary operators in C# have predefined implementations. In addition, user-defined implementations can be introduced by including **operator declarations** in classes and structs. User-defined operator implementations **always take precedence** over predefined operator implementations; only when no applicable user-defined operator exists are the predefined implementations considered. An operator declaration is a `public static` member function using the `operator` keyword, named `operator X` where `X` is the operator symbol. Unary operators take one parameter; binary operators take two. At least one parameter must be of the containing type (or a type parameter constrained to it). Operator declarations **cannot modify the syntax, precedence, or associativity** of an operator. When a binary operator is overloaded, the corresponding compound assignment operator is implicitly overloaded (e.g., `+` also enables `+=`). The assignment operator `=` itself cannot be overloaded. Conversion operators (`implicit` and `explicit`) provide user-defined type conversions and are declared with the `operator` keyword.

**Beginner-Friendly Explanation**
Operator overloading lets your own types **use the same symbols as numbers and strings**. Imagine you create a `Money` type. Without operator overloading, adding two amounts requires writing `Money.Add(a, b)`. With operator overloading, you can define what `+` means for `Money`, so you can simply write `a + b`. It makes your types intuitive and expressive. The catch is that you must be **consistent and predictable**: `+` should add, `==` should test equality, and `<` should test ordering. If you make `+` subtract, other developers will be confused. Overload operators sparingly, only when your type genuinely behaves like a primitive value.

---

### Key Characteristics

- **Predefined operators can be overloaded** — Unary (`+`, `-`, `!`, `~`, `++`, `--`, `true`, `false`) and binary (`+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`, `>>>`, `==`, `!=`, `<`, `>`, `<=`, `>=`) operators can be overloaded.
- **Some operators cannot be overloaded** — `=`, `&&`, `||`, `[]`, `()`, `.`, `->`, `??`, `?:`, `=>`, `new`, `typeof`, `is`, `as`, `sizeof`, `checked`, `unchecked`, `default`, and others are excluded.
- **Comparison operators must be overloaded in pairs** — `==` with `!=`, `<` with `>`, and `<=` with `>=`.
- **Compound assignment is implicit** — Overloading `+` also enables `+=`; overloading `*` also enables `*=`.
- **Conversion operators use `implicit` and `explicit`** — User-defined conversions supplement the built-in conversions.
- **Static by requirement** — Operator declarations must be `public static` (with a C# 14 exception for `++` and `--` as instance methods).
- **Cannot change precedence or associativity** — The operator’s grammatical role is fixed.

---

### Prerequisites

- Understanding of C# classes, structs, and `static` members.
- Familiarity with the predefined operators and their usual meanings.
- Knowledge of `Equals` and `GetHashCode` (required when overloading `==`).
- Awareness of implicit and explicit reference/numeric conversions.
- Basic understanding of operator precedence and associativity.

---

### Related Programming Areas

- **Domain modelling** — Value types like `Money`, `Temperature`, `Complex`, and `Vector` benefit from operator overloading.
- **Mathematical libraries** — Matrices, vectors, and complex numbers use arithmetic operators extensively.
- **Equality and hashing** — Overloading `==` and `!=` connects to `Equals` and `GetHashCode`.
- **API design** — Framework Design Guidelines recommend operator overloading only for primitive-like types.
- **Interoperability** — Conversion operators simplify integration between custom types and built-in types.

---

### Core Concepts / Features

1. Operator Overloading (Overarching Concept)
2. Overloadable Operators
3. Non-Overloadable Operators
4. Overloading Rules
5. Comparison Operators (`==`, `!=`, `<`, `>`, `<=`, `>=`)
6. Arithmetic Operators (`+`, `-`, `*`, `/`, `%`)
7. Conversion Operators (`implicit` and `explicit`)
8. Operator Overloading Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Operator Overloading (Overarching Concept)

### Definitions

**Core Definition**
Operator overloading is the mechanism by which a user-defined type provides its own implementation of a predefined C# operator, enabling instances of that type to be used with familiar operator syntax.

**Technical Definition**
C# allows user-defined types to overload operators by defining **static member functions** using the `operator` keyword. The method must be named `operator X`, where `X` is the operator symbol. Unary operators have one parameter; binary operators have two. In each case, at least one parameter must be of the containing type (or a type parameter constrained to it). User-defined operator implementations take precedence over predefined implementations. The `operator` keyword is also used to declare conversion operators (`implicit` and `explicit`).

**Beginner-Friendly Explanation**
Operator overloading is **teaching your type new tricks with familiar symbols**. If you have a `Fraction` type, you can teach `+` to add fractions. Then `a + b` works just like it does for `int` or `double`. It makes your type feel native to C#.

---

### Purposes

- To make domain types feel like primitive built-in types.
- To provide intuitive syntax for mathematical and logical operations.
- To enable custom types to participate in expressions naturally.
- To reduce verbose method calls (`a.Add(b)`) in favour of operators (`a + b`).
- To support symmetric operations and comparison semantics.

---

### Syntax Rules and Structure

```
public   static   ReturnType   operator   op(Parameters)
{
    // implementation
}
```

**Component breakdown**

- `public static` — Operator declarations must be public and static (C# 14 adds an instance option for `++`/`--`).
- `ReturnType` — The result type.
- `operator` — The keyword introducing the operator declaration.
- `op` — The operator symbol (e.g., `+`, `==`).
- `Parameters` — Unary: one parameter; binary: two parameters; at least one must be of the containing type.

#### Syntax Rules

- At least one parameter must be of the containing type.
- Unary operators take one parameter; binary operators take two.
- Comparison operators must be overloaded in pairs.
- Overloading a binary operator implicitly overloads its compound assignment form.
- Operator declarations cannot change precedence, associativity, or syntax.

#### Constraints and Limitations

- Not all operators can be overloaded (see Section 3).
- Cannot overload `=`, `&&`, `||`, `[]`, `()`, or the conditional operator.
- Should not throw exceptions from operator implementations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Overloading `+` for a `Complex` Type

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
        var a = new Complex(1, 2);
        var b = new Complex(3, 4);
        Console.WriteLine(a + b);   // 4 + 6i
    }
}
```

**Expected Output**

```
4 + 6i
```

**Why this output occurs**

- The `+` operator is overloaded to add the real and imaginary parts of two `Complex` values.

---

#### Example 2: Overloading `==` and `!=`

```csharp
using System;

public readonly struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    public static bool operator ==(Point a, Point b) => a.X == b.X && a.Y == b.Y;
    public static bool operator !=(Point a, Point b) => !(a == b);

    public override bool Equals(object? obj) => obj is Point p && this == p;
    public override int GetHashCode() => HashCode.Combine(X, Y);
}

class Program
{
    static void Main()
    {
        var p1 = new Point(1, 2);
        var p2 = new Point(1, 2);
        Console.WriteLine(p1 == p2);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- The overloaded `==` compares `X` and `Y`; `!=` is defined as the negation of `==`.

---

### Real-World Cases

- **Mathematical types** — `Complex`, `Vector`, `Matrix`, `Fraction`.
- **Financial types** — `Money` with `+`, `-`, `==`.
- **Physical units** — `Temperature`, `Distance`, `Speed`.
- **Date/time** — `DateTime` supports `+` and `-` with `TimeSpan`.

---

### References

- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading
- Expressions — C# language specification (operator overloading) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1243-operator-overloading

---

## 2. Overloadable Operators

### Definitions

**Core Definition**
Overloadable operators are the predefined C# operators that user-defined types are permitted to redefine.

**Technical Definition**
The overloadable unary operators are `+`, `-`, `!`, `~`, `++`, `--`, `true`, and `false`. The overloadable binary operators are `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`, `>>>`, `==`, `!=`, `>`, `<`, `<=`, and `>=`. The `true` and `false` operators must be overloaded together. Comparison operators must be overloaded in pairs (`==`/`!=`, `<`/`>`, `<=`/`>=`). When a binary operator is overloaded, the corresponding compound assignment operator is implicitly overloaded.

**Beginner-Friendly Explanation**
These are the symbols you are **allowed to teach your type**. You can redefine how `+`, `-`, `*`, `/`, `==`, `<`, and many others work for your own classes and structs.

---

### Purposes

- To define arithmetic behaviour for numeric-like types.
- To define equality and ordering for value-like types.
- To define bitwise operations for flag-like types.
- To define increment/decrement semantics for counter-like types.
- To define truthiness for types used in Boolean contexts.

---

### Syntax Rules and Structure (Table of Overloadable Operators)

| Category | Operators |
|---|---|
| **Unary** | `+x`, `-x`, `!x`, `~x`, `++`, `--`, `true`, `false` |
| **Binary arithmetic** | `x + y`, `x - y`, `x * y`, `x / y`, `x % y` |
| **Binary bitwise** | `x & y`, `x | y`, `x ^ y`, `x << y`, `x >> y`, `x >>> y` |
| **Binary comparison** | `x == y`, `x != y`, `x < y`, `x > y`, `x <= y`, `x >= y` |

**Component breakdown**

- `true` / `false` — Must be overloaded together; enable Boolean contexts.
- `++` / `--` — Can be overloaded; C# 14 allows instance methods.
- `>>>` — Unsigned right shift; overloadable with `op_UnsignedRightShift` in C# 11+.

#### Syntax Rules

- All listed operators can be overloaded.
- Comparison operators must be overloaded in pairs.
- `true` and `false` must be overloaded together.
- Compound assignments are implicitly overloaded.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Overloading Unary `-` and `++`

```csharp
using System;

public struct Counter
{
    public int Value;

    public static Counter operator -(Counter c) => new Counter { Value = -c.Value };
    public static Counter operator ++(Counter c) => new Counter { Value = c.Value + 1 };
}

class Program
{
    static void Main()
    {
        var c = new Counter { Value = 5 };
        Console.WriteLine((-c).Value);   // -5
        Console.WriteLine((++c).Value);  // 6
    }
}
```

**Expected Output**

```
-5
6
```

**Why this output occurs**

- Unary `-` negates the value; `++` increments it.

---

### Real-World Cases

- **Flags enums** — Overloading `&`, `|`, `^` for permission types.
- **Counters** — Overloading `++` and `--`.
- **Numeric types** — Overloading arithmetic operators.

---

### References

- Overloadable operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#overloadable-operators
- Expressions — C# language specification (overloadable unary and binary operators) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1243-operator-overloading

---

## 3. Non-Overloadable Operators

### Definitions

**Core Definition**
Non-overloadable operators are C# operators that cannot be redefined by user-defined types.

**Technical Definition**
The following operators cannot be overloaded: `=`, `.`, `?.`, `?[]`, `->`, `=>`, `??`, `?:`, `&&`, `||`, `[]`, `()`, `new`, `typeof`, `is`, `as`, `sizeof`, `checked`, `unchecked`, `default`, and `nameof`. The assignment operator `=` cannot be overloaded, but compound assignments like `+=` are evaluated using the corresponding binary operator (`+`), which can be overloaded. The conditional logical operators `&&` and `||` cannot be overloaded directly, but they are evaluated using `&` and `|`, which can be overloaded. The array indexing operator `[]` cannot be overloaded, but indexers can be defined. The cast operator `()` cannot be overloaded, but conversion operators (`implicit` and `explicit`) can be defined.

**Beginner-Friendly Explanation**
Some symbols are **off-limits**. You cannot change how `=` assigns, how `.` accesses members, how `&&` short-circuits, or how `new` creates objects. These operators are tied to the fundamental semantics of the language.

---

### Purposes

- To preserve the core semantics of the language.
- To prevent dangerous or confusing redefinitions of assignment and control flow.
- To ensure that `&&` and `||` retain their short-circuiting behaviour.
- To keep member access and object creation predictable.

---

### Syntax Rules and Structure (Table of Non-Overloadable Operators)

| Category | Operators |
|---|---|
| **Assignment** | `=` |
| **Member access** | `.`, `?.`, `?[]`, `->` |
| **Conditional** | `?:` |
| **Logical** | `&&`, `||` |
| **Null-related** | `??`, `??=` |
| **Indexing** | `[]` (use indexers instead) |
| **Cast** | `()` (use conversion operators instead) |
| **Object creation** | `new` |
| **Type testing** | `is`, `as`, `typeof`, `sizeof` |
| **Others** | `=>`, `checked`, `unchecked`, `default`, `nameof` |

**Component breakdown**

- `&&` / `||` — Cannot be overloaded, but `&` and `|` can, enabling `&&`/`||` for custom types.
- `[]` — Cannot be overloaded, but indexers can be defined.
- `()` — Cannot be overloaded as a cast, but `implicit`/`explicit` conversion operators can be defined.

#### Syntax Rules

- These operators cannot be redefined.
- Some have alternative mechanisms (indexers, conversion operators).

---

### Annotated Code Examples and Expected Outputs

#### Example: Alternative to Overloading `[]` — Indexers

```csharp
using System;

public class Matrix
{
    private readonly double[,] _data = new double[3, 3];

    public double this[int row, int col]
    {
        get => _data[row, col];
        set => _data[row, col] = value;
    }
}

class Program
{
    static void Main()
    {
        var m = new Matrix();
        m[0, 0] = 1.0;
        Console.WriteLine(m[0, 0]);   // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- The indexer provides element access syntax without overloading the `[]` operator.

---

### Real-World Cases

- **Indexers** — Collections and matrices use indexers instead of `[]` overloading.
- **Conversion operators** — Types define `implicit`/`explicit` conversions instead of `()` overloading.

---

### References

- Non-overloadable operators — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#non-overloadable-operators
- Expressions — C# language specification (non-overloadable operators) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1243-operator-overloading

---

## 4. Overloading Rules

### Definitions

**Core Definition**
Overloading rules are the syntactic and semantic constraints that operator declarations must satisfy.

**Technical Definition**
An operator declaration must be `public` and `static` (C# 14 adds an instance option for `++` and `--`). Unary operators take one parameter; binary operators take two. At least one parameter must be of the containing type (or a type parameter constrained to it). Comparison operators must be overloaded in pairs (`==`/`!=`, `<`/`>`, `<=`/`>=`). The `true` and `false` operators must be overloaded together. When a binary operator is overloaded, the corresponding compound assignment operator is implicitly overloaded. Operator declarations cannot modify precedence, associativity, or syntax.

**Beginner-Friendly Explanation**
These are the **rules of the game**: your operator method must be public and static, it must take the right number of parameters, at least one parameter must be your type, and comparison operators must come in pairs.

---

### Purposes

- To ensure operator declarations are consistent and discoverable.
- To enforce symmetry in comparison operations.
- To guarantee that operators are associated with a type.
- To preserve the language’s grammatical rules.
- To prevent ambiguity in operator resolution.

---

### Syntax Rules and Structure

```
public   static   ReturnType   operator   op(Parameters)
```

**Component breakdown**

- `public` — Required accessibility.
- `static` — Required modifier (except C# 14 instance `++`/`--`).
- Unary: 1 parameter; binary: 2 parameters.
- At least one parameter of the containing type.

#### Syntax Rules

- Must be `public static`.
- Correct parameter count.
- At least one parameter of the containing type.
- Comparison operators in pairs.
- `true`/`false` together.
- Cannot change precedence or associativity.

#### Constraints and Limitations

- Cannot overload non-overloadable operators.
- Cannot throw exceptions from operators.
- Cannot change the operator’s grammatical role.

---

### Annotated Code Examples and Expected Outputs

#### Example: Correctly Declared Operator

```csharp
using System;

public struct Fraction
{
    public int Numerator { get; }
    public int Denominator { get; }

    public Fraction(int numerator, int denominator)
    {
        Numerator = numerator;
        Denominator = denominator;
    }

    public static Fraction operator +(Fraction a, Fraction b) =>
        new Fraction(
            a.Numerator * b.Denominator + b.Numerator * a.Denominator,
            a.Denominator * b.Denominator);

    public static bool operator ==(Fraction a, Fraction b) =>
        a.Numerator * b.Denominator == b.Numerator * a.Denominator;

    public static bool operator !=(Fraction a, Fraction b) => !(a == b);

    public override bool Equals(object? obj) => obj is Fraction f && this == f;
    public override int GetHashCode() => HashCode.Combine(Numerator, Denominator);
}
```

**Expected Output (conceptual)**

```
// Correctly declared public static operators; comparison pair present
```

**Why this is correct**

- `public static` modifiers.
- `+` takes two `Fraction` parameters.
- `==` and `!=` overloaded as a pair.
- `Equals` and `GetHashCode` overridden.

---

### Real-World Cases

- **All operator overloads** — The rules apply universally.
- **Library design** — Following rules ensures operators work predictably.

---

### References

- Operator overloading — C# reference (rules) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading
- Expressions — C# language specification (operator declarations) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1243-operator-overloading

---

## 5. Comparison Operators (`==`, `!=`, `<`, `>`, `<=`, `>=`)

### Definitions

**Core Definition**
Comparison operators are overloadable operators that test equality or ordering between two operands. They must be overloaded in pairs.

**Technical Definition**
The comparison operators `==`, `!=`, `<`, `>`, `<=`, and `>=` can be overloaded. If `==` is overloaded, `!=` must also be overloaded, and vice versa. If `<` is overloaded, `>` must also be overloaded, and similarly for `<=` and `>=`. When overloading `==`, it is strongly recommended to also override `Equals` and `GetHashCode` to maintain consistency with hash-based collections. The `==` operator should compare for equality and return `bool`; implementations that produce unexpected results are strongly discouraged.

**Beginner-Friendly Explanation**
Comparison operators let you say **“are these equal?”** and **“which is bigger?”** for your type. You must always implement them in pairs: `==` with `!=`, `<` with `>`, `<=` with `>=`. This ensures that your type behaves consistently.

---

### Purposes

- To define value equality for value-like types.
- To define ordering for sortable types.
- To enable use in collections and LINQ.
- To support natural syntax for comparisons.
- To integrate with `Equals` and `GetHashCode`.

---

### Syntax Rules and Structure

```
public   static   bool   operator   ==(T   a,   T   b)   {   ...   }
public   static   bool   operator   !=(T   a,   T   b)   =>   !(a   ==   b);
```

**Component breakdown**

- `==` / `!=` — Equality pair.
- `<` / `>` — Ordering pair.
- `<=` / `>=` — Ordering pair.
- Return type: `bool`.

#### Syntax Rules

- Must be overloaded in pairs.
- Return `bool`.
- Should be consistent with `Equals` and `GetHashCode`.
- Should not throw exceptions.

#### Constraints and Limitations

- Cannot overload only one of a pair.
- Inconsistent implementations can break collections.

---

### Annotated Code Examples and Expected Outputs

#### Example: Equality and Ordering

```csharp
using System;

public readonly struct Version : IComparable<Version>
{
    public int Major { get; }
    public int Minor { get; }

    public Version(int major, int minor) => (Major, Minor) = (major, minor);

    public static bool operator ==(Version a, Version b) =>
        a.Major == b.Major && a.Minor == b.Minor;

    public static bool operator !=(Version a, Version b) => !(a == b);

    public static bool operator <(Version a, Version b) =>
        a.Major < b.Major || (a.Major == b.Major && a.Minor < b.Minor);

    public static bool operator >(Version a, Version b) => b < a;

    public static bool operator <=(Version a, Version b) => !(b < a);

    public static bool operator >=(Version a, Version b) => !(a < b);

    public int CompareTo(Version other) => this < other ? -1 : this > other ? 1 : 0;

    public override bool Equals(object? obj) => obj is Version v && this == v;
    public override int GetHashCode() => HashCode.Combine(Major, Minor);
}

class Program
{
    static void Main()
    {
        var v1 = new Version(1, 2);
        var v2 = new Version(1, 3);
        Console.WriteLine(v1 < v2);    // True
        Console.WriteLine(v1 == v2);    // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- `<` compares major and minor versions; `==` compares both fields.

---

### Real-World Cases

- **Versioning** — Comparing semantic versions.
- **Money** — Comparing amounts.
- **Dates** — Comparing dates.

---

### References

- Comparison operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#comparison-operators
- Equality operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators

---

## 6. Arithmetic Operators (`+`, `-`, `*`, `/`, `%`)

### Definitions

**Core Definition**
Arithmetic operators are overloadable operators that perform addition, subtraction, multiplication, division, and remainder on user-defined types.

**Technical Definition**
The binary arithmetic operators `+`, `-`, `*`, `/`, and `%` can be overloaded for user-defined types. The unary `+` and `-` operators can also be overloaded. When a binary arithmetic operator is overloaded, the corresponding compound assignment operator (`+=`, `-=`, `*=`, `/=`, `%=`) is implicitly overloaded. Overloading arithmetic operators should follow the intuitive mathematical meaning of the operator.

**Beginner-Friendly Explanation**
These operators let your type **do math**. You can define how `+` adds two `Money` values or how `*` scales a `Vector` by a number.

---

### Purposes

- To provide natural arithmetic for numeric-like types.
- To enable compound assignment implicitly.
- To support mathematical algorithms and formulas.
- To make domain types expressive.

---

### Syntax Rules and Structure

```
public   static   T   operator   +(T   a,   T   b)   {   ...   }
public   static   T   operator   -(T   a,   T   b)   {   ...   }
public   static   T   operator   *(T   a,   T   b)   {   ...   }
public   static   T   operator   /(T   a,   T   b)   {   ...   }
public   static   T   operator   %(T   a,   T   b)   {   ...   }
```

**Component breakdown**

- `+` — Addition.
- `-` — Subtraction.
- `*` — Multiplication.
- `/` — Division.
- `%` — Remainder.

#### Syntax Rules

- Binary operators take two parameters.
- At least one parameter of the containing type.
- Compound assignment implicitly overloaded.

#### Constraints and Limitations

- Should not throw exceptions.
- Should follow intuitive meaning.

---

### Annotated Code Examples and Expected Outputs

#### Example: Overloading `+` and `*` for `Money`

```csharp
using System;

public readonly struct Money
{
    public decimal Amount { get; }

    public Money(decimal amount) => Amount = amount;

    public static Money operator +(Money a, Money b) => new Money(a.Amount + b.Amount);
    public static Money operator *(Money a, decimal factor) => new Money(a.Amount * factor);

    public override string ToString() => Amount.ToString("C");
}

class Program
{
    static void Main()
    {
        var price = new Money(19.99m);
        var total = price * 3;
        Console.WriteLine(total);   // $59.97
    }
}
```

**Expected Output**

```
$59.97
```

**Why this output occurs**

- `*` multiplies the amount by a factor.

---

### Real-World Cases

- **Money arithmetic** — Addition and scaling.
- **Vector math** — Addition and scalar multiplication.
- **Fractions** — Addition and multiplication.

---

### References

- Arithmetic operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators
- Operator overloading — C# reference (arithmetic) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading

---

## 7. Conversion Operators (`implicit` and `explicit`)

### Definitions

**Core Definition**
Conversion operators are user-defined operators that define how a type can be converted to or from another type. They are declared with `implicit` or `explicit`.

**Technical Definition**
A user-defined type can define a custom **implicit** or **explicit** conversion from or to another type, provided a standard conversion does not already exist between the same two types. The declaration uses the `operator` keyword with either `implicit` or `explicit`. Implicit conversions occur automatically and must never lose information or throw exceptions. Explicit conversions require a cast expression. The source or destination type must be the containing type.

**Beginner-Friendly Explanation**
Conversion operators let you **smoothly convert** between your type and other types. `implicit` means the conversion happens automatically (like `int` to `double`). `explicit` means you must write a cast (like `double` to `int`).

---

### Purposes

- To integrate custom types with built-in types.
- To allow seamless assignment and method calls.
- To control when conversions are safe (implicit) vs. require acknowledgement (explicit).
- To avoid information loss in implicit conversions.

---

### Syntax Rules and Structure

```
public   static   implicit   operator   TargetType(SourceType   value)   {   ...   }
public   static   explicit   operator   TargetType(SourceType   value)   {   ...   }
```

**Component breakdown**

- `implicit` — Automatic conversion.
- `explicit` — Requires a cast.
- `TargetType` — The type being converted to.
- `SourceType` — The type being converted from.

#### Syntax Rules

- Must be `public static`.
- Source or target must be the containing type.
- Implicit conversions must never lose data or throw.
- Explicit conversions require a cast.

#### Constraints and Limitations

- Cannot define a conversion if a standard conversion already exists.
- `is` and `as` operators do not consider user-defined conversions.

---

### Annotated Code Examples and Expected Outputs

#### Example: Implicit and Explicit Conversion

```csharp
using System;

public readonly struct Digit
{
    private readonly byte _digit;

    public Digit(byte digit)
    {
        if (digit > 9) throw new ArgumentOutOfRangeException(nameof(digit));
        _digit = digit;
    }

    public static implicit operator byte(Digit d) => d._digit;
    public static explicit operator Digit(byte b) => new Digit(b);

    public override string ToString() => $"{_digit}";
}

class Program
{
    static void Main()
    {
        var d = new Digit(7);
        byte number = d;          // implicit: no cast
        Console.WriteLine(number); // 7

        Digit digit = (Digit)number; // explicit: cast required
        Console.WriteLine(digit);    // 7
    }
}
```

**Expected Output**

```
7
7
```

**Why this output occurs**

- `byte` to `Digit` is implicit because it never loses data (the constructor validates).
- `Digit` to `byte` is explicit because the cast must be written.

---

### Real-World Cases

- **Numeric wrappers** — Converting between `Money` and `decimal`.
- **Identifiers** — Converting between `Guid` and `string`.
- **Units** — Converting between `Kilometers` and `Meters`.

---

### References

- User-defined conversion operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/user-defined-conversion-operators
- Conversion operators — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#conversion-operators

---

## 8. Operator Overloading Best Practices

### Definitions

**Core Definition**
Operator overloading best practices are the design guidelines that ensure overloaded operators are intuitive, symmetric, safe, and consistent with the framework’s conventions.

**Technical Definition**
Framework Design Guidelines recommend **avoiding operator overloads except in types that should feel like primitive built-in types**. When overloading, do so **symmetrically** (overload related operators together), provide **friendly-named methods** with equivalent functionality, and **never throw exceptions** from operator implementations. Comparison operators must be overloaded in pairs, and `==` should be consistent with `Equals` and `GetHashCode`. Implicit conversions must never lose information or throw.

**Beginner-Friendly Explanation**
Best practices are the habits that keep operator overloading from becoming confusing: only overload operators on types that really feel like numbers or strings, always overload related operators together, never throw from an operator, and provide named methods as alternatives.

---

### Purposes

- To keep operator overloads intuitive and predictable.
- To ensure symmetry and consistency.
- To prevent exceptions from operators.
- To avoid misuse of operator syntax.
- To provide alternative named methods for other languages.

---

### Guidelines and Structure

#### General Guidelines

```
// Only overload operators on primitive-like types
public readonly struct Money
{
    public static Money operator +(Money a, Money b) => ...;
    public static Money Add(Money a, Money b) => a + b; // friendly name
}

// Overload comparison operators in pairs
public static bool operator ==(Point a, Point b) => ...;
public static bool operator !=(Point a, Point b) => !(a == b);

// Never throw from operators
public static Money operator /(Money a, decimal divisor) =>
    divisor == 0 ? Money.Zero : new Money(a.Amount / divisor);

// Implicit conversions must not lose data
public static implicit operator decimal(Money m) => m.Amount;
```

**Component breakdown**

- **Primitive-like types** — Overload only when the type represents a value.
- **Symmetry** — Overload pairs together.
- **Friendly names** — Provide `Add`, `Equals`, `CompareTo`.
- **No exceptions** — Operators should be total functions.
- **Safe implicit conversions** — Never lose data.

#### Rules and Constraints

- ❌ **Avoid** operator overloads except for primitive-like types.
- ✔️ **Consider** operator overloads for primitive-like types.
- ✔️ **Overload symmetrically**.
- ✔️ **Provide friendly-named methods**.
- ❌ **Do not throw exceptions** from operators.
- ❌ **Do not** provide operator overloads unless at least one operand is of the defining type.
- ❌ **Do not** overload `==` without overriding `Equals` and `GetHashCode`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Symmetric Operators with Friendly Names

```csharp
using System;

public readonly struct Vector2
{
    public double X { get; }
    public double Y { get; }

    public Vector2(double x, double y) => (X, Y) = (x, y);

    public static Vector2 operator +(Vector2 a, Vector2 b) =>
        new(a.X + b.X, a.Y + b.Y);

    public static Vector2 operator +(Vector2 a, double scalar) =>
        new(a.X + scalar, a.Y + scalar);

    public static Vector2 operator +(double scalar, Vector2 a) =>
        a + scalar;   // symmetric

    public static Vector2 Add(Vector2 a, Vector2 b) => a + b;   // friendly name

    public static bool operator ==(Vector2 a, Vector2 b) =>
        a.X == b.X && a.Y == b.Y;

    public static bool operator !=(Vector2 a, Vector2 b) => !(a == b);

    public override bool Equals(object? obj) => obj is Vector2 v && this == v;
    public override int GetHashCode() => HashCode.Combine(X, Y);
}
```

**Expected Output (conceptual)**

```
// Symmetric operators, friendly name, no exceptions, consistent equality
```

**Why this is good practice**

- `+` is defined for `Vector2 + Vector2`, `Vector2 + double`, and `double + Vector2`.
- `Add` provides a named alternative.
- `==` and `!=` are paired; `Equals` and `GetHashCode` are overridden.

---

### Real-World Cases

- **Framework types** — `DateTime`, `TimeSpan`, `decimal` follow these guidelines.
- **Domain models** — `Money`, `Temperature`, `Vector` benefit from intuitive operators.
- **Library design** — Following guidelines ensures operators work predictably across languages.

---

### References

- Operator Overloads — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/operator-overloads
- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Instance `++`/`--` operators | Introduced in **C# 14**; earlier versions require `static`. |
| `>>>` overloading | Overloadable in **C# 11+**; earlier versions do not have the operator. |
| User-defined compound assignment | **C# 14** allows explicit overloads; earlier versions implicitly overload. |
| `true`/`false` overloading | Must be overloaded together; used in Boolean contexts. |
| Implicit conversions | Must never lose data or throw; **strongly recommended**. |
| Operator overloading on reference types | Allowed, but **use caution**; equality semantics can be subtle. |
| `==` without `Equals`/`GetHashCode` | **Violates guidelines**; breaks hash-based collections. |

---

## References (Consolidated)

- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading
- Overloadable operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#overloadable-operators
- Non-overloadable operators — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#non-overloadable-operators
- Comparison operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#comparison-operators
- Arithmetic operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators
- User-defined conversion operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/user-defined-conversion-operators
- Conversion operators — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#conversion-operators
- Operator Overloads — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/operator-overloads
- Expressions — C# language specification (operator overloading) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1243-operator-overloading
- Equality operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators

---