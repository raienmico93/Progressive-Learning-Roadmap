# Java Sealed Types: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** A sealed type is a class or interface that restricts which other classes or interfaces may extend or implement it, giving the author of the type explicit control over its permitted subtypes.

**Technical Definition:** Sealed classes and interfaces, finalized in JDK 17 by JEP 409, are a Java language feature that allows the author of a class or interface to control which code is responsible for implementing it. A class is sealed by applying the `sealed` modifier to its declaration. After any `extends` and `implements` clauses, a `permits` clause specifies the classes or interfaces that are permitted to extend or implement the sealed type. A sealed class or interface can be extended or implemented only by those classes and interfaces permitted to do so. The permitted subclasses must be declared with one of three modifiers: `final`, `sealed`, or `non-sealed`. Sealed types provide a foundation for exhaustive analysis in pattern matching, allowing the compiler to guarantee that all possible subtypes are covered in a `switch` expression without requiring a `default` clause.

**Beginner-Friendly Explanation:** Normally in Java, any class can extend another class unless it is `final`. Sealed classes flip this around: you explicitly list which classes are allowed to extend you. Imagine you are designing a graphics library with a `Shape` class. You want only `Circle`, `Square`, and `Rectangle` to be shapes — you don't want someone else coming along and creating a `Triangle` class that extends `Shape`. With a sealed class, you write `sealed class Shape permits Circle, Square, Rectangle` and Java enforces that rule. This makes your code safer and enables the compiler to check that you have handled every possible shape in your code.

### Key Characteristics

Sealed types were first previewed in JDK 15 (JEP 360), refined in JDK 16 (JEP 397), and finalized in JDK 17 (JEP 409). A sealed class or interface restricts which other classes or interfaces may extend or implement it. Every permitted subclass must be declared with exactly one of three modifiers: `final` (closes the hierarchy), `sealed` (continues the controlled hierarchy), or `non-sealed` (opens the subclass to unrestricted extension). Permitted subclasses must reside in the same module as the sealed type (if the sealed type is in a named module) or in the same package (if the sealed type is in the unnamed module). The `permits` clause may be omitted if all permitted subclasses are declared in the same source file as the sealed type. Sealed types are a foundation for exhaustive pattern matching: a `switch` expression over a sealed type is exhaustive without a `default` clause if it covers all permitted subtypes.

### Prerequisites

Readers should be familiar with Java classes, interfaces, inheritance, and access modifiers. A basic understanding of `final` classes and pattern matching (including `instanceof` patterns and `switch` expressions) is helpful for the sections on exhaustiveness and pattern matching. Familiarity with Java records is beneficial for understanding the synergy between sealed types and records.

### Related Programming Areas with Explanation

Sealed types are related to data-oriented programming (where they model sum types alongside records as product types), pattern matching (where they enable exhaustive analysis), design patterns (particularly the State pattern and Visitor pattern, where sealed hierarchies provide controlled extensibility), and API design (where sealed types provide fine-grained inheritance control). They are also closely related to algebraic data types (ADTs) from functional programming, where sealed interfaces model sum types and records model product types.

## Core Concepts and Key Features

### 1. Declaration Mechanics

**Core Definition:** The declaration mechanics of sealed types involve the `sealed` modifier, the `permits` clause, and the rules for implicit permits when subclasses are in the same source file.

**Technical Definition:** To seal a class or interface, add the `sealed` modifier to its declaration. Then, after any `extends` and `implements` clauses, add the `permits` clause, which specifies the classes that may extend the sealed class. If the permitted subclasses are declared in the same source file as the sealed type, the `permits` clause may be omitted, and the compiler infers the permitted subclasses from the declarations in the same compilation unit.

**Beginner-Friendly Explanation:** You declare a sealed class by writing `sealed class Name permits Sub1, Sub2`. If you put all the subclasses in the same file, you can skip the `permits` part and just write `sealed class Name`.

#### 1.1 Sealed Classes and Sealed Interfaces

**Definitions**

- **Core Definition:** A sealed class or interface is a type whose set of direct subtypes is explicitly restricted by its author.
- **Technical Definition:** A class is sealed by applying the `sealed` modifier to its declaration. A sealed class or interface can be extended or implemented only by those classes and interfaces permitted to do so. The `sealed` modifier is a contextual keyword and is not reserved. A sealed interface is declared with the `sealed` modifier in the interface declaration.
- **Beginner-Friendly Explanation:** A sealed class or interface is like a VIP club — only the listed members are allowed in. You write `sealed` before `class` or `interface`, and then list who is allowed to extend or implement it.

**Purposes**

- To control which code is responsible for implementing a class or interface.
- To provide a more declarative way than access modifiers to restrict the use of a superclass.
- To support exhaustive analysis in pattern matching by providing a known, finite set of subtypes.

**Syntax Structures and Rules**

Complete general syntax:

```java
public sealed class ClassName permits Sub1, Sub2 { }
public sealed interface InterfaceName permits Sub1, Sub2 { }
```

Component breakdown:

- `sealed`: the modifier that restricts the hierarchy.
- `permits`: the clause that lists the allowed subtypes.
- `Sub1, Sub2`: the permitted direct subclasses or subinterfaces.

Syntax rules: A sealed class must have at least one permitted subclass. The `permits` clause must list all direct subclasses if present. The permitted subclasses must be in the same module (or package, if in the unnamed module) as the sealed type.

Constraints and limitations: A sealed class cannot be instantiated directly if it is abstract. A sealed class may be abstract or concrete. A sealed interface may be implemented by records, classes, or other interfaces.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic sealed class**

```java
// Step 1: Declare a sealed class with permitted subclasses
public sealed class Shape permits Circle, Square, Rectangle { }

// Step 2: Define the permitted subclasses
final class Circle extends Shape {
    double radius;
    Circle(double radius) { this.radius = radius; }
}

final class Square extends Shape {
    double side;
    Square(double side) { this.side = side; }
}

sealed class Rectangle extends Shape permits FilledRectangle {
    double length, width;
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
}

final class FilledRectangle extends Rectangle {
    int red, green, blue;
    FilledRectangle(double length, double width, int red, int green, int blue) {
        super(length, width);
        this.red = red; this.green = green; this.blue = blue;
    }
}

// Step 3: Use the sealed hierarchy
public class ShapeExample {
    public static void main(String[] args) {
        Shape circle = new Circle(5);
        System.out.println("Created: " + circle.getClass().getSimpleName());
    }
}
```

Expected Output:

```
Created: Circle
```

**Why the code produces this result:** `Circle` is a permitted subclass of `Shape`, so it can extend `Shape`. `Rectangle` is also permitted and is itself sealed, permitting `FilledRectangle`. The hierarchy is controlled.

**Example 2: Sealed interface**

```java
public sealed interface Shape permits Circle, Square { }

record Circle(double radius) implements Shape { }
record Square(double side) implements Shape { }

public class ShapeInterfaceExample {
    public static void main(String[] args) {
        Shape s1 = new Circle(5);
        Shape s2 = new Square(4);

        System.out.println(s1);
        System.out.println(s2);
    }
}
```

Expected Output:

```
Circle[radius=5.0]
Square[side=4.0]
```

**Real-World Cases with Explanation**

Sealed classes are used in graphics libraries to restrict shape types, in financial systems to model a fixed set of transaction types, and in compilers to represent a fixed set of AST node types.

**References Links**

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Sealed Classes (Java Language Updates) – https://docs.oracle.com/en/java/javase/24/language/sealed-classes-and-interfaces.html

#### 1.2 The `permits` Clause

**Definitions**

- **Core Definition:** The `permits` clause is an explicit list of the classes or interfaces that are allowed to directly extend or implement a sealed type.
- **Technical Definition:** The `permits` clause appears after any `extends` and `implements` clauses in a class or interface declaration. It specifies the classes that may extend the sealed class. The `permits` clause can list a class or interface by its canonical name. If the `permits` clause is present, it must list all direct subclasses. If the `permits` clause is absent, the permitted subclasses are inferred from the declarations in the same compilation unit.
- **Beginner-Friendly Explanation:** The `permits` clause is the guest list for your VIP club. You write `permits Sub1, Sub2` and only those types are allowed to extend or implement your sealed type.

**Purposes**

- To explicitly declare which types are allowed to extend or implement a sealed type.
- To provide a clear, declarative specification of the sealed hierarchy.
- To enable the compiler to verify that all permitted subtypes are accounted for.

**Syntax Structures and Rules**

Complete general syntax:

```java
public sealed class Shape permits com.example.Circle, com.example.Square { }
```

Component breakdown:

- `permits`: the keyword introducing the list.
- `com.example.Circle`: the canonical name of a permitted subclass.

Syntax rules: If the `permits` clause is present, it must list all direct subclasses. It is a compile-time error if a class in the `permits` clause is not in the same module (or package) as the sealed type, or if it does not directly extend the sealed type.

Constraints and limitations: The `permits` clause may be omitted if all permitted subclasses are in the same source file. The `permits` clause cannot be used without the `sealed` modifier.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Explicit `permits` clause**

```java
public sealed class Vehicle permits Car, Truck, Motorcycle { }

final class Car extends Vehicle { }
final class Truck extends Vehicle { }
final class Motorcycle extends Vehicle { }

public class VehicleExample {
    public static void main(String[] args) {
        Vehicle v = new Car();
        System.out.println("Vehicle created: " + v.getClass().getSimpleName());
    }
}
```

Expected Output:

```
Vehicle created: Car
```

**Example 2: Attempting an unpermitted subclass fails**

```java
public sealed class Animal permits Dog, Cat { }
final class Dog extends Animal { }
final class Cat extends Animal { }

// This would cause a compile-time error:
// class Bird extends Animal { }
```

Expected Output: Compilation error: "class is not allowed to extend sealed class".

**Real-World Cases with Explanation**

The `permits` clause is used in API design to explicitly document and enforce which types are allowed to participate in a hierarchy, such as allowing only specific plugin types to extend a base plugin class.

**References Links**

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Sealed Classes (Java Language Updates) – https://docs.oracle.com/en/java/javase/24/language/sealed-classes-and-interfaces.html

#### 1.3 Implicit Permits

**Definitions**

- **Core Definition:** The ability to omit the `permits` clause when all permitted subclasses are declared in the same source file as the sealed type.
- **Technical Definition:** If the `permits` clause is absent, the compiler infers the permitted subclasses from the declarations in the same compilation unit. The compiler synthesizes a `permits` clause containing exactly the list of subtypes declared in the same compilation unit. The permitted subclasses must still satisfy all other constraints (same package/module, accessibility).
- **Beginner-Friendly Explanation:** If you put all your subclasses in the same file as the sealed class, you don't need to write `permits` — Java figures it out automatically.

**Purposes**

- To reduce boilerplate when all subclasses are co-located in one file.
- To simplify the declaration of small sealed hierarchies.
- To allow the compiler to infer the permitted subclasses from context.

**Syntax Structures and Rules**

Complete general syntax:

```java
public sealed class Figure {
    // permits clause omitted
}

final class Circle extends Figure { }
non-sealed class Square extends Figure { }
sealed class Rectangle extends Figure { }
final class FilledRectangle extends Rectangle { }
```

Component breakdown:

- The `sealed` modifier is present.
- The `permits` clause is omitted.
- The permitted subclasses are declared in the same file.

Syntax rules: The compiler infers the permitted subclasses from the declarations in the same compilation unit. The inferred list must include all direct subclasses declared in the file.

Constraints and limitations: This works only if all permitted subclasses are in the same source file. If any permitted subclass is in a different file, the `permits` clause must be explicit.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Implicit permits with all subclasses in one file**

```java
// All in one file: Figure.java
public sealed class Figure {
    // permits omitted; inferred from declarations below
}

final class Circle extends Figure { float radius; }
non-sealed class Square extends Figure { float side; }
sealed class Rectangle extends Figure { float length, width; }
final class FilledRectangle extends Rectangle { int red, green, blue; }

public class FigureExample {
    public static void main(String[] args) {
        Figure f = new Circle();
        System.out.println("Figure: " + f.getClass().getSimpleName());
    }
}
```

Expected Output:

```
Figure: Circle
```

**Real-World Cases with Explanation**

Implicit permits are useful in small, self-contained hierarchies where all subtypes are closely related and naturally co-located, such as a small set of configuration option types.

**References Links**

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Sealed Classes (Java Language Updates) – https://docs.oracle.com/en/java/javase/24/language/sealed-classes-and-interfaces.html

### 2. Subclass Constraints (The Three Options)

**Core Definition:** Every permitted subclass of a sealed type must be declared with exactly one of three modifiers: `final`, `sealed`, or `non-sealed`, which determine how the hierarchy continues.

**Technical Definition:** A class that extends a sealed class must be declared with exactly one of the following modifiers: `final` (cannot be extended), `sealed` (continues the controlled hierarchy), or `non-sealed` (opens the subclass to unrestricted extension). It is a compile-time error if a class with a sealed direct superclass or sealed direct superinterface does not declare either `final`, `sealed`, or `non-sealed`.

**Beginner-Friendly Explanation:** When you are allowed to extend a sealed class, you must choose how the story continues. `final` means the story ends here. `sealed` means you continue the controlled list. `non-sealed` means you open the door for anyone to extend you.

#### 2.1 `final`

**Definitions**

- **Core Definition:** A `final` subclass of a sealed type closes the hierarchy; no further subclasses are allowed.
- **Technical Definition:** A `final` class cannot be extended. When a permitted subclass of a sealed type is declared `final`, it represents a leaf node in the sealed hierarchy.
- **Beginner-Friendly Explanation:** `final` means "this is the end of the line" — nobody can extend this class.

**Purposes**

- To close a branch of the sealed hierarchy permanently.
- To indicate that a subtype is a terminal, complete implementation.
- To support exhaustiveness checking by ensuring the subtype cannot be further extended.

**Syntax Structures and Rules**

Complete general syntax:

```java
final class SubType extends SealedType { }
```

Component breakdown:

- `final`: the modifier that prevents further extension.
- `extends SealedType`: the sealed superclass.

Syntax rules: The `final` modifier must be present on any permitted subclass that does not continue the sealed hierarchy.

Constraints and limitations: A `final` class cannot be extended by any other class.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Final subclass**

```java
public sealed class Payment permits CreditCard, Cash { }
final class CreditCard extends Payment { }
final class Cash extends Payment { }

public class FinalSubclassExample {
    public static void main(String[] args) {
        Payment p = new CreditCard();
        System.out.println("Payment: " + p.getClass().getSimpleName());
    }
}
```

Expected Output:

```
Payment: CreditCard
```

**Real-World Cases with Explanation**

`final` subclasses are used when a particular subtype represents a complete, unchanging concept, such as a `CashPayment` or a `CreditCardPayment` that should never have further subtypes.

**References Links**

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Sealed Classes (Java Language Updates) – https://docs.oracle.com/en/java/javase/24/language/sealed-classes-and-interfaces.html

#### 2.2 `sealed`

**Definitions**

- **Core Definition:** A `sealed` subclass of a sealed type continues the controlled hierarchy with its own nested `permits` clause.
- **Technical Definition:** A permitted subclass declared as `sealed` may have its own `permits` clause listing its permitted subclasses. This allows the hierarchy to continue with controlled extension at deeper levels.
- **Beginner-Friendly Explanation:** `sealed` means "I am also sealed, and here is my own list of who can extend me."

**Purposes**

- To continue the controlled hierarchy at deeper levels.
- To allow multi-level sealed hierarchies with fine-grained control.
- To support complex domain models with nested subtype relationships.

**Syntax Structures and Rules**

Complete general syntax:

```java
sealed class SubType extends SealedType permits SubSubType { }
```

Component breakdown:

- `sealed`: the modifier that continues the controlled hierarchy.
- `permits SubSubType`: the nested list of permitted subclasses.

Syntax rules: A `sealed` subclass must have its own `permits` clause (unless all its permitted subclasses are in the same file). Its permitted subclasses must also be `final`, `sealed`, or `non-sealed`.

Constraints and limitations: The permitted subclasses of a `sealed` subclass must satisfy the same module/package and accessibility constraints.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Multi-level sealed hierarchy**

```java
public sealed class Vehicle permits Car, Truck { }
sealed class Car extends Vehicle permits Sedan, SUV { }
final class Sedan extends Car { }
final class SUV extends Car { }
final class Truck extends Vehicle { }

public class SealedSubclassExample {
    public static void main(String[] args) {
        Vehicle v = new Sedan();
        System.out.println("Vehicle: " + v.getClass().getSimpleName());
    }
}
```

Expected Output:

```
Vehicle: Sedan
```

**Real-World Cases with Explanation**

Multi-level sealed hierarchies are used in domain models where a category has subcategories, such as `Vehicle` → `Car` → `Sedan`/`SUV`, or `Shape` → `Polygon` → `Triangle`/`Rectangle`.

**References Links**

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Sealed Classes (Java Language Updates) – https://docs.oracle.com/en/java/javase/24/language/sealed-classes-and-interfaces.html

#### 2.3 `non-sealed`

**Definitions**

- **Core Definition:** A `non-sealed` subclass of a sealed type breaks out of the control loop and opens the subclass to standard, unrestricted extension.
- **Technical Definition:** A `non-sealed` class is specifically opening up a class to be subclassed. Any class can extend a `non-sealed` class. When a permitted subclass is declared `non-sealed`, it removes the sealed restriction for that branch of the hierarchy.
- **Beginner-Friendly Explanation:** `non-sealed` means "I was allowed to extend the sealed class, but now I am opening my own door — anyone can extend me."

**Purposes**

- To allow controlled extension at one level while opening up unrestricted extension at a deeper level.
- To provide a controlled escape hatch from a sealed hierarchy.
- To support hybrid designs where some branches are sealed and others are open.

**Syntax Structures and Rules**

Complete general syntax:

```java
non-sealed class SubType extends SealedType { }
```

Component breakdown:

- `non-sealed`: the modifier that opens the class to unrestricted extension.
- `extends SealedType`: the sealed superclass.

Syntax rules: A `non-sealed` class can be extended by any class. It does not need a `permits` clause.

Constraints and limitations: The `non-sealed` modifier is a contextual keyword and must be the first modifier on the class declaration.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Non-sealed subclass**

```java
public sealed class Shape permits Circle, Polygon { }
final class Circle extends Shape { }
non-sealed class Polygon extends Shape { }

// Polygon is non-sealed, so any class can extend it
class Triangle extends Polygon { }
class Quadrilateral extends Polygon { }

public class NonSealedExample {
    public static void main(String[] args) {
        Shape s = new Triangle();
        System.out.println("Shape: " + s.getClass().getSimpleName());
    }
}
```

Expected Output:

```
Shape: Triangle
```

**Real-World Cases with Explanation**

`non-sealed` is used in plugin architectures where a base type is sealed to a few categories, but one category is explicitly designed to be extended by third-party plugins.

**References Links**

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Sealed Classes (Java Language Updates) – https://docs.oracle.com/en/java/javase/24/language/sealed-classes-and-interfaces.html

### 3. Exhaustive Type Modeling and Pattern Matching

**Core Definition:** Sealed types enable exhaustive type modeling, where the compiler can verify that all possible subtypes are covered in a `switch` expression, eliminating the need for a `default` branch.

**Technical Definition:** An exhaustive `switch` on a sealed type covers the permitted subtypes the compiler knows about. When the selector is a sealed type, the switch can be made exhaustive by enumerating the permitted subtypes of the sealed type, case by case. No `default` is needed. If a `switch` expression is exhaustive at compile time but not at run time (because the sealed hierarchy has been changed and recompiled), a `MatchException` is thrown.

**Beginner-Friendly Explanation:** Because a sealed type tells the compiler exactly what all the possible subtypes are, the compiler can check that your `switch` statement handles every single one. If you forget one, you get a compile-time error instead of a surprise at runtime.

#### 3.1 Exhaustiveness Checks

**Definitions**

- **Core Definition:** The compiler's ability to verify that a `switch` expression over a sealed type covers all permitted subtypes.
- **Technical Definition:** A `switch` expression must be exhaustive, meaning that the switch must produce a result for every possible non-null value of the selector. When the selector is a sealed type, the switch can be made exhaustive by enumerating the permitted subtypes of the sealed type, case by case.
- **Beginner-Friendly Explanation:** Exhaustiveness means the compiler checks that you have not forgotten any possible case. If you handle all the subtypes of a sealed type, your switch is exhaustive.

**Purposes**

- To guarantee at compile time that all possible subtypes are handled.
- To eliminate the need for a `default` clause in switches over sealed types.
- To provide compile-time safety when new subtypes are added later.

**Syntax Structures and Rules**

Complete general syntax:

```java
String result = switch (sealedValue) {
    case SubType1 s1 -> "handled SubType1";
    case SubType2 s2 -> "handled SubType2";
};
// No default needed if all permitted subtypes are covered
```

Component breakdown:

- `switch (sealedValue)`: the switch expression over a sealed type.
- `case SubType1 s1`: the pattern covering the first permitted subtype.
- No `default` clause is needed if all permitted subtypes are covered.

Syntax rules: If a `switch` statement or expression is exhaustive at compile time but not at run time (because the sealed hierarchy has been changed and recompiled), a `MatchException` is thrown. It is almost never necessary to use `default` when switching over a sealed type.

Constraints and limitations: Exhaustiveness checking works only for sealed types. For non-sealed types, a `default` clause is required.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Exhaustive switch over a sealed interface**

```java
public sealed interface Fruit permits Apple, Orange, Banana { }

record Apple(String variety) implements Fruit { }
record Orange(String variety) implements Fruit { }
record Banana(String variety) implements Fruit { }

public class ExhaustiveSwitchExample {
    public static void main(String[] args) {
        Fruit fruit = new Apple("Fuji");
        String description = switch (fruit) {
            case Apple a -> "Apple: " + a.variety();
            case Orange o -> "Orange: " + o.variety();
            case Banana b -> "Banana: " + b.variety();
        };
        System.out.println(description);
    }
}
```

Expected Output:

```
Apple: Fuji
```

**Why the code produces this result:** The `switch` expression covers all three permitted subtypes of `Fruit` (`Apple`, `Orange`, `Banana`), so it is exhaustive and no `default` is needed.

**Example 2: Compile-time error when a subtype is missing**

```java
public sealed interface Fruit permits Apple, Orange, Banana { }

record Apple(String variety) implements Fruit { }
record Orange(String variety) implements Fruit { }
record Banana(String variety) implements Fruit { }

public class MissingCaseExample {
    public static void main(String[] args) {
        Fruit fruit = new Apple("Fuji");
        // This would cause a compile-time error:
        // String description = switch (fruit) {
        //     case Apple a -> "Apple: " + a.variety();
        //     case Orange o -> "Orange: " + o.variety();
        //     // Banana missing!
        // };
    }
}
```

Expected Output: Compilation error: "the switch expression does not cover all possible input values".

**Real-World Cases with Explanation**

Exhaustive switches over sealed types are used in data processing pipelines where every variant of a data structure must be handled, in compilers where every AST node type must be processed, and in financial systems where every transaction type must be accounted for.

**References Links**

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Preparing for Change: Safe Switching over Sealed APIs – https://openjdk.org/projects/amber/guides/exhaustiveness-guide

#### 3.2 No `default` Branch Required

**Definitions**

- **Core Definition:** The elimination of the `default` clause in switches over sealed types when all permitted subtypes are covered.
- **Technical Definition:** A `default` label is permitted, but not required, in the case where the switch block exhausts all the permitted direct subclasses and subinterfaces of an abstract sealed class or sealed interface. It is almost never necessary to use `default` when switching over a sealed type.
- **Beginner-Friendly Explanation:** When you switch over a sealed type, you don't need to write a `default` case — the compiler already knows you have covered everything.

**Purposes**

- To eliminate boilerplate `default` clauses.
- To prevent the `default` clause from hiding missing cases.
- To enable compile-time detection of new subtypes when the sealed hierarchy evolves.

**Syntax Structures and Rules**

Complete general syntax:

```java
String result = switch (sealedValue) {
    case SubType1 s1 -> "...";
    case SubType2 s2 -> "...";
    // No default!
};
```

Component breakdown:

- No `default` clause is present.
- The switch is exhaustive because all permitted subtypes are covered.

Syntax rules: Adding a `default` label hides the developer's intent that the switch is exhaustive and deprives the compiler of the ability to check for missing cases.

Constraints and limitations: If the sealed hierarchy is later extended with a new subtype, the switch will fail to compile if recompiled, alerting the developer to the change.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: No default branch needed**

```java
public sealed interface Notification permits Email, SMS, Push { }
record Email(String address, String subject) implements Notification { }
record SMS(String phoneNumber, String message) implements Notification { }
record Push(String deviceToken, String payload) implements Notification { }

public class NoDefaultExample {
    public static void main(String[] args) {
        Notification n = new Email("a@b.com", "Hello");
        String channel = switch (n) {
            case Email e -> "Email to " + e.address();
            case SMS s -> "SMS to " + s.phoneNumber();
            case Push p -> "Push to " + p.deviceToken();
        };
        System.out.println(channel);
    }
}
```

Expected Output:

```
Email to a@b.com
```

**Real-World Cases with Explanation**

Eliminating `default` branches is used in notification systems, event processing systems, and any domain where the set of message types is fixed and should be exhaustively handled.

**References Links**

- Preparing for Change: Safe Switching over Sealed APIs – https://openjdk.org/projects/amber/guides/exhaustiveness-guide
- JEP 409: Sealed Classes – https://openjdk.org/jeps/409

#### 3.3 Pattern Matching for Switch (Java 21+)

**Definitions**

- **Core Definition:** The combination of sealed types with pattern matching in `switch` expressions, allowing safe deconstruction of permitted types.
- **Technical Definition:** Pattern matching for `switch`, finalized in JEP 441 (Java 21), allows `switch` to test patterns against the selector expression. When combined with sealed types, pattern matching enables exhaustive analysis of all permitted subtypes without a `default` clause. The patterns can include type patterns, record patterns, and guards.
- **Beginner-Friendly Explanation:** Pattern matching for `switch` lets you match against different subtypes and extract their data at the same time. Combined with sealed types, it gives you a safe, exhaustive way to handle every possible variant of your data.

**Purposes**

- To safely deconstruct permitted types in `switch` expressions.
- To combine subtype checking with data extraction in a single construct.
- To enable declarative, data-oriented programming with algebraic data types.

**Syntax Structures and Rules**

Complete general syntax:

```java
String result = switch (sealedValue) {
    case SubType1(Type1 field1, Type2 field2) -> "..." + field1 + field2;
    case SubType2(Type3 field3) -> "..." + field3;
};
```

Component breakdown:

- `case SubType1(Type1 field1, Type2 field2)`: a record pattern that matches `SubType1` and extracts its components.
- `case SubType2(Type3 field3)`: a record pattern that matches `SubType2`.

Syntax rules: The patterns must cover all permitted subtypes for the switch to be exhaustive. Guards (`when` clauses) can be used to add conditions.

Constraints and limitations: Record patterns work with records. For non-record classes, type patterns can be used without deconstruction.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Pattern matching with sealed types and records**

```java
public sealed interface Shape permits Circle, Rectangle, Triangle { }
record Circle(double radius) implements Shape { }
record Rectangle(double length, double width) implements Shape { }
record Triangle(double base, double height) implements Shape { }

public class PatternMatchingExample {
    public static void main(String[] args) {
        Shape shape = new Circle(5);
        double area = switch (shape) {
            case Circle c -> Math.PI * c.radius() * c.radius();
            case Rectangle r -> r.length() * r.width();
            case Triangle t -> 0.5 * t.base() * t.height();
        };
        System.out.printf("Area: %.2f%n", area);
    }
}
```

Expected Output:

```
Area: 78.54
```

**Why the code produces this result:** The `switch` uses record patterns to match each permitted subtype and extract its components. The `Circle` pattern extracts the `radius` component, which is used to compute the area.

**Real-World Cases with Explanation**

Pattern matching with sealed types is used in expression evaluators, where each expression type (literal, binary operation, unary operation) is a sealed subtype, and the evaluator uses a switch to handle each case.

**References Links**

- JEP 441: Pattern Matching for switch – https://openjdk.org/jeps/441
- Pattern Matching for switch (Java Language Updates) – https://docs.oracle.com/en/java/javase/21/language/pattern-matching-switch.html

### 4. Ecosystem Integration and Rules

**Core Definition:** Sealed types have specific integration rules with the module system, package structure, and other Java language features such as records.

**Technical Definition:** Permitted subclasses must reside within the same module as the sealed type (if the sealed type is in a named module) or in the same package (if the sealed type is in the unnamed module). Records are implicitly final, making them ideal candidates for permitted subclasses.

**Beginner-Friendly Explanation:** Sealed types have rules about where their subclasses can live: they must be in the same module or package. Records work perfectly as permitted subclasses because they are already final.

#### 4.1 Same Package/Module Restriction

**Definitions**

- **Core Definition:** The rule that permitted subclasses of a sealed type must be in the same module (if in a named module) or same package (if in the unnamed module) as the sealed type.
- **Technical Definition:** If a sealed class C belongs to a named module, then every class named in the `permits` clause of the declaration of C must belong to the same module as C; otherwise a compile-time error occurs. If a sealed class C belongs to an unnamed module, then every class named in the `permits` clause must belong to the same package as C. Permitted subclasses must also be accessible by the sealed class at compile time.
- **Beginner-Friendly Explanation:** You cannot put a permitted subclass in a different module or package than the sealed type. They have to be neighbors.

**Purposes**

- To maintain encapsulation and package-level access control.
- To ensure that the sealed hierarchy is self-contained within a module or package.
- To support the module system's strong encapsulation guarantees.

**Syntax Structures and Rules**

Complete general syntax:

```java
// In module com.example:
public sealed class Shape permits Circle, Square { }
// Circle and Square must also be in module com.example
```

Component breakdown:

- The sealed type is in a named module.
- All permitted subclasses must be in the same named module.

Syntax rules: The permitted subclasses must be accessible by the sealed class at compile time. They must directly extend the sealed class.

Constraints and limitations: This restriction prevents cross-module sealed hierarchies, which is intentional to support strong encapsulation.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Same package restriction**

```java
package com.example.shapes;

public sealed class Shape permits Circle, Square { }

// In the same package:
package com.example.shapes;
public final class Circle extends Shape { }

// In a different package, this would fail:
// package com.example.other;
// public final class Circle extends Shape { } // COMPILE ERROR
```

Expected Output: Compilation error if the subclass is in a different package.

**Real-World Cases with Explanation**

The same package/module restriction is used in library design to keep sealed hierarchies self-contained, ensuring that all subtypes are maintained together and can be safely evolved.

**References Links**

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Sealed Classes (Java Language Updates) – https://docs.oracle.com/en/java/javase/24/language/sealed-classes-and-interfaces.html

#### 4.2 Synergy with Java Records

**Definitions**

- **Core Definition:** Records are implicitly final, making them ideal, zero-boilerplate candidates for permitted subclasses of sealed interfaces.
- **Technical Definition:** Records cannot extend another class, but they can implement sealed interfaces. Because records are implicitly final, they naturally satisfy the requirement that a permitted subclass be declared `final`, `sealed`, or `non-sealed`. Records and sealed interfaces together model algebraic data types: sealed interfaces are sum types and records are product types.
- **Beginner-Friendly Explanation:** Records are perfect partners for sealed types. Because records are already final, you can use them directly as permitted subclasses without any extra modifiers. Together they let you model data like a pro.

**Purposes**

- To reduce boilerplate when creating sealed hierarchies with data-carrying subtypes.
- To model algebraic data types (sum types with product types) in Java.
- To enable data-oriented programming with immutable data carriers.

**Syntax Structures and Rules**

Complete general syntax:

```java
public sealed interface Person permits Student, Teacher { }
record Student(String name, int id) implements Person { }
record Teacher(String name, String subject) implements Person { }
```

Component breakdown:

- `sealed interface Person permits Student, Teacher`: the sealed sum type.
- `record Student(String name, int id) implements Person`: a product type that is a permitted subtype.

Syntax rules: Records automatically satisfy the `final` requirement for permitted subclasses. The record's components become the data carried by the subtype.

Constraints and limitations: Records cannot extend classes, so they can only be used as permitted subclasses of sealed interfaces, not sealed classes.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Sealed interface with record implementations**

```java
public sealed interface Person permits Student, Teacher { }
record Student(String name, int id) implements Person { }
record Teacher(String name, String subject) implements Person { }

public class RecordSealedExample {
    public static void main(String[] args) {
        Person p = new Student("Alice", 101);
        String info = switch (p) {
            case Student s -> "Student " + s.name() + " (ID: " + s.id() + ")";
            case Teacher t -> "Teacher " + t.name() + " teaches " + t.subject();
        };
        System.out.println(info);
    }
}
```

Expected Output:

```
Student Alice (ID: 101)
```

**Real-World Cases with Explanation**

Sealed interfaces with record implementations are used in event systems where each event type is a record (`UserCreated`, `OrderPlaced`, `PaymentProcessed`), in expression trees where each expression type is a record, and in API response modeling where each response variant is a record.

**References Links**

- Data-Oriented Programming in Java – https://inside.java/2023/03/09/data-oriented-programming/
- When are sealed classes and records used together in Java? – https://stackoverflow.com/questions/62421148/when-are-sealed-classes-and-records-used-together-in-java

## Summary Table of Key Features

| Feature | Description | Key Benefit |
|---|---|---|
| `sealed` modifier | Restricts which types can extend/implement | Controlled inheritance |
| `permits` clause | Lists allowed subtypes | Explicit hierarchy definition |
| Implicit permits | Omit `permits` if subclasses are in same file | Reduced boilerplate |
| `final` subclass | Closes the hierarchy | No further extension |
| `sealed` subclass | Continues the controlled hierarchy | Multi-level control |
| `non-sealed` subclass | Opens the subclass to unrestricted extension | Controlled escape hatch |
| Exhaustiveness | Compiler verifies all subtypes are covered | Compile-time safety |
| No `default` needed | Switch over sealed type is exhaustive without `default` | Cleaner code, no hidden bugs |
| Pattern matching | Deconstruct sealed types in `switch` | Declarative data processing |
| Same package/module | Permitted subclasses must be co-located | Encapsulation |
| Records synergy | Records are ideal permitted subclasses | Zero-boilerplate ADTs |

## Deprecated, Unsafe, and Version-Specific Notes

- Sealed classes were previewed in JDK 15 (JEP 360) and JDK 16 (JEP 397) and finalized in JDK 17 (JEP 409).
- Pattern matching for `switch` was finalized in JDK 21 (JEP 441). Record patterns were finalized in JDK 21 (JEP 440).
- The `permits` clause may be omitted if all permitted subclasses are in the same source file. However, this makes the hierarchy less discoverable, and explicit `permits` clauses are recommended for public APIs.
- Adding a `default` clause to a switch over a sealed type is permitted but discouraged, as it hides the developer's intent and deprives the compiler of the ability to check for missing cases.
- If a sealed hierarchy is extended with a new subtype and a client switch is not recompiled, a `MatchException` is thrown at runtime when the new subtype is encountered. This is by design: it provides feedback that the program's assumptions are no longer valid.
- Permitted subclasses of sealed types cannot be in a different module or package. This is a deliberate restriction to support strong encapsulation.

## References

- JEP 409: Sealed Classes – https://openjdk.org/jeps/409
- Sealed Classes (Java Language Updates, Java SE 24) – https://docs.oracle.com/en/java/javase/24/language/sealed-classes-and-interfaces.html
- Preparing for Change: Safe Switching over Sealed APIs – https://openjdk.org/projects/amber/guides/exhaustiveness-guide
- JEP 441: Pattern Matching for switch – https://openjdk.org/jeps/441
- Pattern Matching for switch (Java Language Updates) – https://docs.oracle.com/en/java/javase/21/language/pattern-matching-switch.html
- Data-Oriented Programming in Java – https://inside.java/2023/03/09/data-oriented-programming/
- Sealed types (OpenJDK Amber Spec Observers) – https://mail.openjdk.org/pipermail/amber-spec-observers/2018-November/000944.html
- Experience with sealed classes & the "same package" rule (OpenJDK) – https://mail.openjdk.org/pipermail/amber-spec-experts/2020-May/001234.html
- When are sealed classes and records used together in Java? – https://stackoverflow.com/questions/62421148/when-are-sealed-classes-and-records-used-together-in-java
- JEP 360: Sealed Classes (Preview) – https://openjdk.org/jeps/360
- JEP 397: Sealed Classes (Second Preview) – https://openjdk.org/jeps/397