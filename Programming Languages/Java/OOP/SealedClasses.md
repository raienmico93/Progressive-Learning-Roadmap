# Java Sealed Classes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A sealed class or interface is a reference type that restricts which other classes or interfaces may extend or implement it, explicitly declaring the complete set of permitted subtypes.

**Technical Definition:** Sealed classes and interfaces, finalized in JDK 17 via JEP 409, enhance the Java language by restricting which other classes or interfaces may extend or implement them. A sealed type declares its permitted direct subtypes through the `permits` clause, and every permitted subtype must be declared `final`, `sealed`, or `non-sealed`, ensuring the inheritance hierarchy is explicit and closed at every level. A sealed class imposes three key constraints on its permitted subclasses: they must belong to the same module (or same package if in an unnamed module), they must directly extend the sealed class, and they must declare exactly one of the three permitted inheritance modifiers.

**Beginner-Friendly Explanation:** A sealed class is like a VIP club with a guest list. Only the people on the list (the permitted subclasses) can get in. Anyone else trying to extend the class is turned away with a compilation error. This is useful when you know exactly what kinds of subclasses a class should have and you want to prevent anyone else from creating new ones.

### Key Characteristics

- **Controlled Inheritance:** Only explicitly permitted classes or interfaces may extend or implement a sealed type.
- **Exhaustive Analysis:** Sealed hierarchies enable the compiler to verify that `switch` expressions and pattern matching are exhaustive without requiring a `default` clause.
- **Three Subtype Modifiers:** Permitted subtypes must be declared `final` (no further extension), `sealed` (continued restriction), or `non-sealed` (reopening extension).
- **Same-Module/Package Constraint:** Permitted subclasses must reside in the same module (for modular code) or same package (for non-modular code) as the sealed class.
- **Declarative Alternative to Access Modifiers:** Sealed classes provide a more declarative way to restrict the use of a superclass than access modifiers alone.
- **Bridge to Algebraic Data Types:** Sealed types bring Java closer to ADT-style modeling, where a base type has a finite, enumerated set of variants.

### Prerequisites

- Java 17 or later (sealed classes were finalized in JDK 17)
- Basic understanding of Java classes, inheritance, and interfaces
- Familiarity with the `final` keyword and access modifiers
- Awareness of pattern matching and `switch` expressions (for exhaustiveness benefits)

### Related Programming Areas

- **Pattern Matching:** Sealed classes provide the foundation for exhaustive pattern analysis in `switch` expressions and `instanceof`.
- **Algebraic Data Types (ADTs):** Sealed hierarchies model sum types, where a value can be one of a fixed set of variants.
- **Domain Modeling:** Sealed classes are ideal for modeling closed domains such as payment statuses, workflow states, and event types.
- **API Design:** Library authors use sealed classes to control which classes are responsible for implementing their interfaces.
- **Records:** Records are implicitly final and often used as permitted subclasses of sealed interfaces.

---

## 1. Sealed Classes

### Definitions

**Core Definition:** A sealed class is a class declared with the `sealed` modifier that restricts which other classes may extend it.

**Technical Definition:** Per JLS §8.1.1.1, a class is sealed by applying the `sealed` modifier to its declaration. The permitted subclasses are specified in a `permits` clause that follows any `extends` or `implements` clauses. A sealed class cannot be instantiated if it is also abstract, but a non-abstract sealed class can be instantiated.

**Beginner-Friendly Explanation:** A sealed class is a class that says: “Only these specific classes are allowed to extend me.” It’s like a parent who only allows certain children to inherit the family business.

### Purposes

- To restrict the set of classes that can extend a class, preventing arbitrary extension
- To model closed domain hierarchies where the set of variants is known and fixed
- To provide the compiler with enough information for exhaustive pattern matching
- To offer a declarative alternative to access modifiers for controlling inheritance

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public sealed class ClassName permits Sub1, Sub2, Sub3 {
    // class body
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `sealed` | Modifier indicating the class restricts extension. |
| `permits` | Clause specifying the permitted direct subclasses. |
| `Sub1, Sub2, Sub3` | The explicitly named permitted subclasses. |

**Syntax Rules:**

- The `permits` clause is optional if all permitted subclasses are declared in the same source file as the sealed class.
- Permitted subclasses must directly extend the sealed class.
- Permitted subclasses must be declared `final`, `sealed`, or `non-sealed`.

**Constraints and Limitations:**

- A sealed class cannot also be `final` (final classes cannot be extended at all).
- The `permits` clause cannot name a class that is not accessible at compile time.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Sealed Class**

```java
// File: Shape.java
// Sealed class declaring its permitted subclasses
public sealed class Shape permits Circle, Square, Rectangle {
    // Common fields and methods
}

// File: Circle.java
public final class Circle extends Shape {
    public float radius;
}

// File: Square.java
public non-sealed class Square extends Shape {
    public double side;
}

// File: Rectangle.java
public sealed class Rectangle extends Shape permits FilledRectangle {
    public double length, width;
}

// File: FilledRectangle.java
public final class FilledRectangle extends Rectangle {
    public int red, green, blue;
}
```

**Expected Output:** (No output; these are declarations only.)

**Step-by-Step Explanation:**

1. `Shape` is declared `sealed` and permits `Circle`, `Square`, and `Rectangle`.
2. `Circle` is `final`, so it cannot be extended further.
3. `Square` is `non-sealed`, meaning it reopens the hierarchy — any class can extend `Square`.
4. `Rectangle` is itself `sealed`, continuing the restriction by permitting only `FilledRectangle`.
5. `FilledRectangle` is `final`, ending the hierarchy.

### Real-World Cases

- **Payment Statuses:** `sealed interface PaymentStatus permits Authorized, Captured, Refunded, Failed` models a fixed set of payment states.
- **Graphics Libraries:** A `Shape` sealed class permits only `Circle`, `Square`, and `Rectangle`.
- **Compiler ASTs:** Abstract syntax tree node types are modeled as sealed hierarchies.

---

## 2. Java 17 Sealed Classes

### Definitions

**Core Definition:** Sealed classes were finalized as a permanent feature in Java 17 (JDK 17) through JEP 409.

**Technical Definition:** JEP 409 finalized sealed classes in JDK 17 with no changes from JDK 16. Sealed classes were first proposed by JEP 360 and delivered in JDK 15 as a preview feature, then refined and delivered again in JDK 16 as a second preview feature.

**Beginner-Friendly Explanation:** Sealed classes became official in Java 17. Before that, they were available as a preview feature in Java 15 and 16, but you had to enable preview features to use them.

### Purposes

- To provide a stable, permanent feature for controlling inheritance in Java
- To enable exhaustive pattern matching in `switch` expressions
- To support domain modeling with closed hierarchies

### Syntax Rules and Structure

**Availability:**

| JDK Version | Status |
|-------------|--------|
| JDK 15 | Preview (JEP 360) |
| JDK 16 | Second Preview (JEP 397) |
| JDK 17 | Final (JEP 409) |

**Syntax Rules:**

- The syntax is the same as described in Section 1.
- No preview flag is required in JDK 17 and later.

**Constraints and Limitations:**

- Requires JDK 17 or later for production use.

### Annotated Code Example

```java
// File: Java17SealedDemo.java
public sealed interface Celestial permits Planet, Star, Comet {
    double distanceFromEarth();
}

final class Planet implements Celestial {
    public double distanceFromEarth() { return 1.0; }
}

final class Star implements Celestial {
    public double distanceFromEarth() { return 1000.0; }
}

final class Comet implements Celestial {
    public double distanceFromEarth() { return 50.0; }
}
```

**Expected Output:** (No output; declaration only.)

**Step-by-Step Explanation:**

1. The sealed interface `Celestial` restricts implementation to `Planet`, `Star`, and `Comet`.
2. Each permitted class is `final`, preventing further extension.
3. This models a closed set of celestial object kinds.

### Real-World Cases

- **Enterprise Applications:** Sealed classes are used for state machines and event hierarchies.
- **API Design:** Library authors use sealed classes to prevent external extension.

---

## 3. sealed keyword

### Definitions

**Core Definition:** The `sealed` keyword is a contextual keyword that marks a class or interface as restricting which other types may extend or implement it.

**Technical Definition:** The `sealed` modifier is applied to a class or interface declaration to indicate that extension/implementation is restricted to a specified set of permitted subtypes. It is a contextual keyword, meaning it is only treated as a keyword in class or interface declarations.

**Beginner-Friendly Explanation:** `sealed` is the keyword that tells Java: “This class has a closed guest list.” Only the classes listed in the `permits` clause can extend it.

### Purposes

- To declare a class or interface as sealed
- To restrict inheritance to a known set of subtypes
- To enable exhaustive pattern matching

### Syntax Rules and Structure

```java
public sealed class ClassName permits Sub1, Sub2 { ... }
public sealed interface InterfaceName permits Impl1, Impl2 { ... }
```

**Syntax Rules:**

- The `sealed` modifier is placed before the `class` or `interface` keyword.
- It is a contextual keyword; it is not reserved in other contexts.

**Constraints and Limitations:**

- A sealed class must have at least one permitted subclass (though this can be implicit if subclasses are in the same file).

### Annotated Code Example

```java
// File: SealedKeywordDemo.java
public sealed class Vehicle permits Car, Truck, Motorcycle {
    // ...
}
```

**Step-by-Step Explanation:**

1. `sealed` marks `Vehicle` as restricted.
2. `permits` lists the three allowed subclasses.
3. No other class may extend `Vehicle`.

### Real-World Cases

- **`java.lang.constant` package:** Uses sealed interfaces for constant descriptors.
- **JDK Internal APIs:** Sealed classes are used to control extension in internal hierarchies.

---

## 4. permits clause

### Definitions

**Core Definition:** The `permits` clause is a syntactic construct that explicitly lists the classes or interfaces permitted to extend or implement a sealed type.

**Technical Definition:** The `permits` clause follows any `extends` or `implements` clauses in a sealed class or interface declaration. It specifies the direct subclasses or subinterfaces that are permitted to extend or implement the sealed type. If all permitted subclasses are declared in the same source file as the sealed class, the `permits` clause may be omitted.

**Beginner-Friendly Explanation:** The `permits` clause is the guest list. It names exactly who is allowed to extend the sealed class.

### Purposes

- To explicitly name the permitted subtypes of a sealed type
- To make the inheritance hierarchy self-documenting
- To enable the compiler to verify exhaustiveness

### Syntax Rules and Structure

```java
public sealed class ClassName permits Sub1, Sub2, Sub3 { ... }
```

**Syntax Rules:**

- The `permits` clause is optional if all subclasses are in the same file.
- Permitted classes must be accessible at compile time.
- Permitted classes must directly extend the sealed class.

**Constraints and Limitations:**

- A sealed class cannot have an empty `permits` clause (unless all subclasses are in the same file).

### Annotated Code Example

```java
// File: PermitsClauseDemo.java
public sealed class Figure permits Circle, Square {
    // ...
}

// Alternatively, if subclasses are in the same file:
public sealed class Figure {
    // permits clause omitted
}

final class Circle extends Figure { }
non-sealed class Square extends Figure { }
```

**Expected Output:** (No output; declaration only.)

**Step-by-Step Explanation:**

1. In the first example, `permits Circle, Square` explicitly names the allowed subclasses.
2. In the second example, the `permits` clause is omitted because `Circle` and `Square` are in the same file as `Figure`.

### Real-World Cases

- **Library APIs:** Explicitly listing permitted subclasses documents the extension points.
- **Domain Models:** The `permits` clause serves as a manifest of all domain variants.

---

## 5. non-sealed keyword

### Definitions

**Core Definition:** The `non-sealed` keyword is a contextual keyword used to declare that a permitted subclass of a sealed class reopens the hierarchy, allowing any class to extend it.

**Technical Definition:** A permitted subclass of a sealed class may be declared `non-sealed`, which means that part of the hierarchy reverts to being open for extension by unknown subclasses. A `non-sealed` class does not restrict who can extend it.

**Beginner-Friendly Explanation:** `non-sealed` is like saying: “The sealed class allowed me in, but I’m opening the door for everyone else.” It breaks the seal at that point in the hierarchy.

### Purposes

- To reopen a portion of a sealed hierarchy for arbitrary extension
- To allow mixing closed and open inheritance within the same hierarchy
- To provide flexibility when some branches of a hierarchy need to be extensible

### Syntax Rules and Structure

```java
public non-sealed class SubclassName extends SealedClass { ... }
```

**Syntax Rules:**

- `non-sealed` is a contextual keyword (a sequence of tokens, not a single identifier).
- It can only be applied to a permitted subclass of a sealed class.
- A `non-sealed` class does not have a `permits` clause.

**Constraints and Limitations:**

- A `non-sealed` class cannot be `final` or `sealed`.

### Annotated Code Example

```java
// File: NonSealedDemo.java
public sealed class Animal permits Dog, Cat, Duck { }

public final class Dog extends Animal { }

public sealed class Cat extends Animal permits PersianCat { }

public non-sealed class Duck extends Animal {
    // Any class can now extend Duck
}
```

**Expected Output:** (No output; declaration only.)

**Step-by-Step Explanation:**

1. `Animal` is sealed and permits `Dog`, `Cat`, and `Duck`.
2. `Dog` is final — no further extension.
3. `Cat` is sealed — it continues the restriction, permitting only `PersianCat`.
4. `Duck` is `non-sealed` — it reopens the hierarchy, allowing any class to extend it.

### Real-World Cases

- **Framework Extension Points:** A sealed base class permits a `non-sealed` adapter class that users can extend freely.
- **Plugin Architectures:** Some branches of the hierarchy are closed (core types), while others are open (plugin types).

---

## 6. final subclasses

### Definitions

**Core Definition:** A `final` subclass of a sealed class is a permitted subclass that cannot be extended any further.

**Technical Definition:** A permitted subclass of a sealed class may be declared `final`, which means it terminates the inheritance hierarchy at that point. No class may extend a `final` class.

**Beginner-Friendly Explanation:** A `final` subclass is the end of the line. It was allowed into the sealed hierarchy, but it cannot be extended by anyone else.

### Purposes

- To terminate a branch of the sealed hierarchy
- To prevent further extension of a specific subtype
- To provide a concrete, immutable implementation

### Syntax Rules and Structure

```java
public final class SubclassName extends SealedClass { ... }
```

**Syntax Rules:**

- A `final` class cannot be extended.
- It must be a direct permitted subclass of a sealed class.

**Constraints and Limitations:**

- A `final` class cannot be `sealed` or `non-sealed`.

### Annotated Code Example

```java
// File: FinalSubclassDemo.java
public sealed class Payment permits CreditCard, PayPal { }

public final class CreditCard extends Payment {
    private final String cardNumber;
    public CreditCard(String cardNumber) { this.cardNumber = cardNumber; }
}

public final class PayPal extends Payment {
    private final String email;
    public PayPal(String email) { this.email = email; }
}
```

**Expected Output:** (No output; declaration only.)

**Step-by-Step Explanation:**

1. `Payment` is sealed and permits two subclasses.
2. Both `CreditCard` and `PayPal` are `final`, so they cannot be extended.
3. This models a fixed set of payment methods.

### Real-World Cases

- **Payment Methods:** Each payment method is a `final` subclass of a sealed `Payment` type.
- **Event Types:** `final` event classes in a sealed `Event` hierarchy.

---

## 7. Sealed Interfaces

### Definitions

**Core Definition:** A sealed interface is an interface declared with the `sealed` modifier that restricts which classes or interfaces may implement or extend it.

**Technical Definition:** Sealed interfaces follow the same rules as sealed classes. The `permits` clause lists the classes or interfaces that are permitted to implement or extend the sealed interface. Permitted implementors must be declared `final`, `sealed`, or `non-sealed`.

**Beginner-Friendly Explanation:** A sealed interface is like a sealed class, but for interfaces. It says: “Only these specific classes can implement me.”

### Purposes

- To restrict implementation of an interface to a known set of classes
- To model closed domain hierarchies with interfaces
- To enable exhaustive pattern matching on interface types

### Syntax Rules and Structure

```java
public sealed interface InterfaceName permits Impl1, Impl2, Impl3 { ... }
```

**Syntax Rules:**

- Same rules as sealed classes apply.
- Permitted implementors must be in the same module or package.

**Constraints and Limitations:**

- A sealed interface cannot be implemented by a class in a different package (unless in the same module).

### Annotated Code Example

```java
// File: SealedInterfaceDemo.java
public sealed interface Item permits Book, Furniture, ElectronicItem {
    String name();
    double price();
}

record Book(String name, double price, String isbn) implements Item { }
record Furniture(String name, double price, String material) implements Item { }
record ElectronicItem(String name, double price, int warrantyMonths) implements Item { }
```

**Expected Output:** (No output; declaration only.)

**Step-by-Step Explanation:**

1. `Item` is a sealed interface permitting three record classes.
2. Each record implements `Item` and provides the required methods.
3. Records are implicitly final, so they terminate the hierarchy.

### Real-World Cases

- **E-Commerce:** A sealed `Item` interface permits `Book`, `Furniture`, and `ElectronicItem`.
- **State Machines:** A sealed `State` interface permits `Idle`, `Running`, and `Stopped`.

---

## 8. Pattern Matching with Sealed Classes

### Definitions

**Core Definition:** Pattern matching with sealed classes allows `switch` expressions and statements to match against all permitted subtypes of a sealed hierarchy, with the compiler verifying exhaustiveness.

**Technical Definition:** A `switch` block is exhaustive for a sealed type if it covers all permitted direct subclasses and subinterfaces. A `default` label is permitted but not required when the switch block exhausts all permitted subtypes.

**Beginner-Friendly Explanation:** Because the compiler knows exactly which subclasses exist, you can write a `switch` statement that covers all of them without needing a `default` case. The compiler checks that you haven’t missed any.

### Purposes

- To write exhaustive pattern matching without a `default` clause
- To ensure all cases are handled at compile time
- To enable safer refactoring when new subtypes are added

### Syntax Rules and Structure

```java
return switch (sealedType) {
    case SubType1 s1 -> ...;
    case SubType2 s2 -> ...;
    case SubType3 s3 -> ...;
    // No default required if exhaustive
};
```

**Syntax Rules:**

- The switch must cover all permitted direct subclasses.
- A `default` label is allowed but not required for exhaustive switches.
- The compiler will report an error if the switch is not exhaustive.

**Constraints and Limitations:**

- Only works when the selector expression’s type is a sealed type.
- Null values are not covered by type patterns and may require handling.

### Annotated Code Example

```java
// File: PatternMatchingSealedDemo.java
public sealed interface Shape permits Circle, Rectangle, Triangle { }

record Circle(double radius) implements Shape { }
record Rectangle(double length, double width) implements Shape { }
record Triangle(double base, double height) implements Shape { }

public class PatternMatchingSealedDemo {
    public static double area(Shape shape) {
        return switch (shape) {
            case Circle c -> Math.PI * c.radius() * c.radius();
            case Rectangle r -> r.length() * r.width();
            case Triangle t -> 0.5 * t.base() * t.height();
            // No default needed — exhaustive!
        };
    }

    public static void main(String[] args) {
        System.out.println("Circle area: " + area(new Circle(5)));
        System.out.println("Rectangle area: " + area(new Rectangle(4, 3)));
        System.out.println("Triangle area: " + area(new Triangle(6, 2)));
    }
}
```

**Expected Output:**
```
Circle area: 78.53981633974483
Rectangle area: 12.0
Triangle area: 6.0
```

**Step-by-Step Explanation:**

1. `Shape` is a sealed interface permitting `Circle`, `Rectangle`, and `Triangle`.
2. The `switch` expression covers all three permitted types.
3. No `default` clause is needed because the switch is exhaustive.
4. If a new permitted subtype were added, the compiler would report an error until the switch was updated.

### Real-World Cases

- **State Machines:** Exhaustive switch on sealed state types.
- **AST Processing:** Exhaustive switch on sealed node types.
- **Payment Processing:** Exhaustive switch on sealed payment statuses.

---

## 9. Exhaustiveness

### Definitions

**Core Definition:** Exhaustiveness in pattern matching means that a `switch` expression or statement covers all possible values of the selector type.

**Technical Definition:** A switch block is exhaustive for a sealed type if it covers all permitted direct subclasses and subinterfaces. The compiler can verify exhaustiveness because sealed types have a known, finite set of subtypes.

**Beginner-Friendly Explanation:** Exhaustiveness means you’ve handled every possible case. With sealed classes, the compiler knows all the cases and can tell you if you’ve missed one.

### Purposes

- To ensure all cases are handled at compile time
- To eliminate the need for a `default` clause
- To catch missing cases when new subtypes are added

### Syntax Rules and Structure

```java
sealed interface I permits A, B, C { }
final class A implements I { }
final class B implements I { }
record C(int j) implements I { }

static int testExhaustive(I i) {
    return switch(i) {
        case A a -> 0;
        case B b -> 1;
        case C c -> 2;
        // No default required!
    };
}
```

**Syntax Rules:**

- The switch must cover all permitted subtypes.
- A `default` label is permitted but not required.
- The compiler reports an error if the switch is not exhaustive.

**Constraints and Limitations:**

- Exhaustiveness only applies to sealed types and certain other closed types (like enums).
- Null values are not covered by type patterns.

### Annotated Code Example

See the example in Section 8.

### Real-World Cases

- **Compilers:** Exhaustive switches on sealed AST node types.
- **Protocol Handlers:** Exhaustive switches on sealed message types.
- **UI State:** Exhaustive switches on sealed screen states.

---

## 10. Sealed Class Best Practices

### Definitions

**Core Definition:** Sealed class best practices are guidelines for designing and using sealed hierarchies effectively.

**Technical Definition:** Best practices include keeping sealed hierarchies focused, documenting the rationale for sealing, using records for data-carrying subtypes, and leveraging exhaustiveness for safe pattern matching.

**Beginner-Friendly Explanation:** Best practices help you use sealed classes in ways that make your code safer and more maintainable.

### Purposes

- To design clean, maintainable sealed hierarchies
- To maximise the benefits of exhaustiveness checking
- To avoid common pitfalls

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Keep hierarchies focused | A sealed hierarchy should model a single, cohesive domain concept. |
| Document the rationale | Explain why the hierarchy is sealed and what each subtype represents. |
| Use records for data | Records are ideal permitted subclasses because they are implicitly final and provide concise data carriers. |
| Leverage exhaustiveness | Write exhaustive switches without `default` clauses to benefit from compiler checking. |
| Prefer sealed interfaces for APIs | Sealed interfaces allow both classes and records to implement, offering more flexibility. |
| Use `non-sealed` sparingly | Opening a hierarchy should be intentional and documented. |
| Ensure same-module/package | All permitted subclasses must reside in the same module or package. |

**Syntax Rules:**

- All permitted subclasses must be in the same module or package as the sealed class.
- Each permitted subclass must be `final`, `sealed`, or `non-sealed`.

**Constraints and Limitations:**

- Sealed classes cannot be used across module boundaries.
- Over-sealing can make a hierarchy rigid and hard to evolve.

### Annotated Code Example

```java
// File: BestPracticesSealed.java
/**
 * Models the result of a payment transaction.
 * Sealed to ensure only these three outcomes are possible.
 */
public sealed interface PaymentResult
    permits PaymentSuccess, PaymentFailure, PaymentPending {

    record PaymentSuccess(String transactionId, double amount)
        implements PaymentResult { }

    record PaymentFailure(String errorCode, String message)
        implements PaymentResult { }

    record PaymentPending(String referenceId)
        implements PaymentResult { }
}

public class BestPracticesSealed {
    public static String describe(PaymentResult result) {
        return switch (result) {
            case PaymentResult.PaymentSuccess s ->
                "Success: " + s.transactionId();
            case PaymentResult.PaymentFailure f ->
                "Failure: " + f.errorCode();
            case PaymentResult.PaymentPending p ->
                "Pending: " + p.referenceId();
            // Exhaustive — no default needed
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(new PaymentResult.PaymentSuccess("TXN-001", 99.99)));
        System.out.println(describe(new PaymentResult.PaymentFailure("ERR-500", "Insufficient funds")));
        System.out.println(describe(new PaymentResult.PaymentPending("REF-123")));
    }
}
```

**Expected Output:**
```
Success: TXN-001
Failure: ERR-500
Pending: REF-123
```

**Step-by-Step Explanation:**

1. `PaymentResult` is a sealed interface permitting three record types.
2. Each record carries data specific to its outcome.
3. The `describe` method uses an exhaustive switch with no `default` clause.
4. The compiler verifies that all cases are covered.

### Real-World Cases

- **Payment Processing:** Sealed hierarchies for payment results, statuses, and methods.
- **Event Systems:** Sealed interfaces for domain events.
- **Workflow Engines:** Sealed types for workflow states and transitions.

---

## References Links

- JEP 409: Sealed Classes - https://openjdk.org/jeps/409
- Sealed Classes and Interfaces — Java Language Updates (Oracle) - https://docs.oracle.com/en/java/javase/27/language/sealed-classes-interfaces.html
- JLS §8.1.1.1: abstract Classes — https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html
- Pattern Matching for switch — Oracle Java Documentation - https://docs.oracle.com/en/java/javase/23/docs/specs/primitive-types-in-patterns-instanceof-switch-jls.html
- Sealed Classes — OpenJDK Specification - https://cr.openjdk.org/~gbierman/jep409/jep409-20210507/specs/sealed-classes-jls.html
- Java Sealed Classes: Complete Guide with Patterns, Examples, and Best Practices — Cleverence - https://www.cleverence.com/articles/oracle-documentation/sealed-classes-java-guide-4621/
- Model Data, the Whole Data, and Nothing but the Data — Inside.java (Oracle) - https://inside.java/2024/05/29/model-data/
- Contextual Keywords — OpenJDK - https://cr.openjdk.org/~gbierman/jep409/jep409-20210507/specs/contextual-keywords-jls.html
- Sealed Classes and Interfaces — Baeldung - https://www.baeldung.com/java-sealed-classes-interfaces
- Java Sealed Classes and Interfaces — Dev.java - https://dev.java/learn/sealed-classes-interfaces/