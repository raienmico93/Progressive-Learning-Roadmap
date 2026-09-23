# Java Polymorphism: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Polymorphism** is the object-oriented programming principle that allows objects of different types to be treated as instances of a common supertype, with each type providing its own specialized behavior for shared operations.

### Technical Definition

Polymorphism in Java is realized through **subtype polymorphism**, where a superclass reference can refer to a subclass object, and the method invoked is determined by the **runtime type** of the object rather than the **compile-time type** of the reference . Java supports two primary forms: **compile-time polymorphism** (static binding via method overloading) and **runtime polymorphism** (dynamic binding via method overriding) . The JVM uses **dynamic method dispatch** to select the most specific implementation at runtime . Type safety during casting is ensured through `instanceof` checks, with modern pattern matching providing more concise and exhaustive alternatives .

### Beginner-Friendly Explanation

Imagine you have a universal remote that can control any TV, DVD player, or sound system. When you press "power," each device responds in its own way—the TV turns on, the DVD player ejects its tray, and the sound system starts playing. You used the same button (same method call), but each device did its own thing. That's polymorphism: one interface, many behaviors.

### Key Characteristics

- **Subtype substitutability**: A subclass object can be used wherever a superclass object is expected.
- **Dynamic dispatch**: Method calls are resolved at runtime based on the actual object type.
- **Two forms**: Compile-time (overloading) and runtime (overriding).
- **Upcasting is safe**: Converting a subtype reference to a supertype is implicit and always valid.
- **Downcasting requires care**: Converting a supertype reference to a subtype requires explicit cast and runtime check.
- **Pattern matching**: Modern Java provides concise, safe downcasting with `instanceof` patterns and exhaustive `switch` .

### Prerequisites

- Basic understanding of classes, inheritance, and the `extends` keyword.
- Familiarity with method overloading and method overriding.
- Knowledge of access modifiers and exception handling in overrides.

### Related Programming Areas

- **Method Overloading**: Compile-time polymorphism.
- **Method Overriding**: Runtime polymorphism.
- **Design Patterns**: Strategy, Template Method, Factory patterns rely on polymorphism.
- **SOLID Principles**: Liskov Substitution Principle (LSP) is foundational to correct polymorphism .

### Core Concepts / Features

1. Compile-Time Polymorphism (Overloading) vs. Runtime Polymorphism (Overriding)
2. Upcasting (Implicit Casting to a Supertype)
3. Downcasting (Explicit Casting to a Subtype)
4. Dynamic Method Dispatch
5. Safe Downcasting: `instanceof` and Pattern Matching
6. Design Principles: Liskov Substitution Principle (LSP) and Coding to an Interface/Type


## Core Concept 1: Compile-Time Polymorphism (Overloading) vs. Runtime Polymorphism (Overriding)

### Definitions

**Core Definition**: **Compile-time polymorphism** (static binding) is achieved through method overloading, where the compiler determines which method to call based on the arguments. **Runtime polymorphism** (dynamic binding) is achieved through method overriding, where the JVM determines which method to call based on the object's actual type.

**Technical Definition**: Method overloading is an example of compile-time/static polymorphism because the compiler resolves the method call at compile time based on the static types of the arguments . Method overriding is an example of runtime/dynamic polymorphism because the JVM resolves the method call at runtime based on the dynamic type of the receiver object . The key distinction is **when** the method is selected: overloaded methods are selected by the compiler, while overridden methods are selected by the JVM.

**Beginner-Friendly Explanation**: Overloading is like a restaurant menu with "small pizza" and "large pizza"—you decide which one to order when you speak (compile time). Overriding is like a franchise where each location makes the "house special" their own way—you don't know which recipe you'll get until you visit the actual restaurant (runtime).

### Purposes

- **Overloading**: To provide convenient variations of a method for different input types or numbers of arguments.
- **Overriding**: To allow subclasses to customize inherited behavior while maintaining a consistent interface.
- To enable both early (compile-time) and late (runtime) binding based on the design need.
- To support flexible, extensible systems where new subclasses can be added without modifying existing code.

### Syntax Rules and Structure

**Compile-Time Polymorphism (Overloading):**

```java
public void process(int value) { }
public void process(String value) { }
public void process(int a, int b) { }
```

**Runtime Polymorphism (Overriding):**

```java
class Superclass {
    public void process() { System.out.println("Super"); }
}

class Subclass extends Superclass {
    @Override
    public void process() { System.out.println("Sub"); }
}
```

**Comparison Table:**

| Aspect | Overloading (Compile-Time) | Overriding (Runtime) |
|--------|---------------------------|----------------------|
| Binding | Static (compile time) | Dynamic (runtime) |
| Selection basis | Argument static types | Object runtime type |
| Method signature | Must differ | Must match |
| Inheritance | Same class (or related) | Superclass/subclass |
| Polymorphic? | Ad-hoc polymorphism | Subtype polymorphism |

**Syntax Rules:**

- Overloading: same method name, different parameter lists .
- Overriding: same signature, `@Override` recommended.
- Overloading is resolved using the **compile-time type** of the receiver and arguments .
- Overriding is resolved using the **runtime type** of the receiver .

**Constraints and Limitations:**

- Cannot overload by return type alone.
- Cannot override `final`, `static`, or `private` methods.
- Overloading can create ambiguity (e.g., with autoboxing/varargs).
- Overriding must respect access, exception, and covariant return rules.

### Annotated Complete Code Examples

**Example 1: Overloading vs. Overriding**

```java
/**
 * Demonstrates compile-time and runtime polymorphism.
 */
class Calculator {
    // Overloaded methods (compile-time polymorphism)
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}

class Animal {
    public void makeSound() {
        System.out.println("Animal makes sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Dog barks");
    }
}

public class PolymorphismDemo {
    public static void main(String[] args) {
        // Compile-time: compiler picks add(int, int)
        Calculator calc = new Calculator();
        System.out.println("add(2,3): " + calc.add(2, 3));
        System.out.println("add(2.5,3.5): " + calc.add(2.5, 3.5));
        System.out.println("add(1,2,3): " + calc.add(1, 2, 3));

        // Runtime: JVM picks Dog.makeSound()
        Animal a = new Dog();
        a.makeSound(); // "Dog barks"
    }
}
```

**Expected Output:**

```
add(2,3): 5
add(2.5,3.5): 6.0
add(1,2,3): 6
Dog barks
```

**Why This Output Occurs:**
- The compiler selects `add(int, int)` for `(2, 3)`, `add(double, double)` for `(2.5, 3.5)`, and `add(int, int, int)` for `(1, 2, 3)` based on argument types .
- `a.makeSound()` dispatches to `Dog.makeSound()` at runtime because `a` refers to a `Dog` object .

### Real-World Cases

- **Collections**: `Collections.sort()` overloaded for `List` and arrays.
- **Template Method Pattern**: Superclass defines the algorithm; subclasses override steps.
- **Strategy Pattern**: Different strategies implement the same interface.

### References

- Dev.java – Polymorphism - https://dev.java/learn/inheritance/polymorphism/ 
- University of Washington – Overloading vs. Overriding - https://courses.cs.washington.edu/courses/cse331/18au/lectures/lect06a-overloading.pdf 


## Core Concept 2: Upcasting (Implicit Casting to a Supertype)

### Definitions

**Core Definition**: **Upcasting** is the implicit conversion of a subclass reference to a superclass reference, allowing a subclass object to be treated as an instance of its superclass.

**Technical Definition**: Upcasting is always safe and implicit because a subclass **is-a** superclass . When a subclass object is assigned to a superclass reference, the compiler allows it without an explicit cast. The reference type determines which methods are **accessible** at compile time, but the runtime type determines which **implementation** executes . This is the foundation of runtime polymorphism.

**Beginner-Friendly Explanation**: Upcasting is like putting a square peg in a round hole that's actually a "shape" hole. Since a square is a shape, you can treat it as a shape without any problem. You lose the ability to call square-specific methods through the shape reference, but you can still call all shape methods.

### Purposes

- To enable polymorphic behavior where different subclasses can be treated uniformly.
- To allow writing generic code that operates on a supertype rather than specific subtypes.
- To support the Liskov Substitution Principle by treating subtypes as supertypes.
- To decouple client code from concrete implementations.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
Superclass reference = new Subclass();  // Implicit upcast
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Superclass` | The compile-time type of the reference. |
| `reference` | The variable holding the reference. |
| `new Subclass()` | The runtime type of the object. |

**Syntax Rules:**

- Upcasting is implicit; no cast operator is needed.
- The subclass must have an "is-a" relationship with the superclass.
- Through the superclass reference, only members declared in the superclass are accessible.
- The runtime type determines which overridden methods execute .

**Constraints and Limitations:**

- Cannot access subclass-specific members through the superclass reference without downcasting.
- Fields are not polymorphic; field access uses the reference type .

### Annotated Complete Code Examples

**Example 1: Upcasting and Polymorphic Behavior**

```java
/**
 * Demonstrates upcasting.
 */
class Shape {
    public void draw() {
        System.out.println("Drawing shape");
    }
}

class Circle extends Shape {
    @Override
    public void draw() {
        System.out.println("Drawing circle");
    }

    public void drawCircle() {
        System.out.println("Circle-specific method");
    }
}

public class UpcastDemo {
    public static void main(String[] args) {
        // Implicit upcast
        Shape shape = new Circle();

        // Calls overridden method (runtime dispatch)
        shape.draw(); // "Drawing circle"

        // Cannot call drawCircle() through Shape reference
        // shape.drawCircle(); // Compile error
    }
}
```

**Expected Output:**

```
Drawing circle
```

**Why This Output Occurs:**
- `Shape shape = new Circle()` upcasts implicitly.
- `shape.draw()` dispatches to `Circle.draw()` at runtime .
- `drawCircle()` is not accessible through the `Shape` reference because it is not declared in `Shape`.

### Real-World Cases

- **Collections**: `List<String> list = new ArrayList<>()` upcasts `ArrayList` to `List`.
- **Event handling**: `EventHandler handler = new ButtonHandler()`.
- **Framework APIs**: Treating all `Shape` objects uniformly for rendering.

### References

- Dev.java – Polymorphism (Virtual Method Invocation) - https://dev.java/learn/inheritance/polymorphism/ 


## Core Concept 3: Downcasting (Explicit Casting to a Subtype)

### Definitions

**Core Definition**: **Downcasting** is the explicit conversion of a superclass reference to a subclass reference, allowing access to subclass-specific members.

**Technical Definition**: Downcasting requires an explicit cast and is not guaranteed to succeed at compile time. If the object's runtime type is not compatible with the target type, a `ClassCastException` is thrown at runtime . Downcasting is necessary when a superclass reference points to a subclass object and you need to access subclass-specific methods or fields. Safe downcasting should be preceded by an `instanceof` check .

**Beginner-Friendly Explanation**: Downcasting is like receiving a package labeled "Shape" and hoping it's actually a "Circle" so you can call circle-specific methods. You have to check first—if it's really a square, trying to treat it as a circle will cause an error.

### Purposes

- To access subclass-specific members through a superclass reference.
- To recover the specific type of an object after upcasting.
- To enable type-specific operations in polymorphic collections.
- To work with legacy APIs that return `Object`.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
Subclass reference = (Subclass) superclassReference;  // Explicit downcast
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Subclass` | The target type of the cast. |
| `(Subclass)` | The explicit cast operator. |
| `superclassReference` | The reference being downcast. |

**Syntax Rules:**

- Downcasting requires an explicit cast operator.
- The cast is checked at runtime; if the object is not an instance of the target type, `ClassCastException` is thrown .
- Safe downcasting should be preceded by `instanceof` check .
- The target type must be a subclass of the reference's compile-time type.

**Constraints and Limitations:**

- Downcasting can fail at runtime if the object's actual type is incompatible.
- `instanceof` should always be used to avoid `ClassCastException`.
- Modern Java recommends pattern matching for `instanceof` to combine check and cast .

### Annotated Complete Code Examples

**Example 1: Unsafe Downcasting (ClassCastException)**

```java
/**
 * Demonstrates unsafe downcasting.
 */
class Animal { }
class Dog extends Animal {
    public void bark() { System.out.println("Woof"); }
}
class Cat extends Animal { }

public class UnsafeDowncastDemo {
    public static void main(String[] args) {
        Animal animal = new Cat();

        // Unsafe downcast: Cat is not a Dog
        try {
            Dog dog = (Dog) animal; // ClassCastException
            dog.bark();
        } catch (ClassCastException e) {
            System.out.println("ClassCastException: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
ClassCastException: class Cat cannot be cast to class Dog
```

**Why This Output Occurs:**
- `animal` refers to a `Cat` object at runtime.
- `(Dog) animal` attempts to downcast to `Dog`, which is incompatible.
- The JVM throws `ClassCastException` at runtime .

### Real-World Cases

- **Event handling**: Casting `Event` to `MouseEvent` to access mouse-specific data.
- **Object deserialization**: Casting `Object` to the expected type.
- **Legacy collections**: Casting elements from a raw `List` to their actual types.

### References

- Dev.java – Polymorphism (Casting Objects) - https://dev.java/learn/inheritance/polymorphism/ 
- GitHub – Safe Downcasting with Pattern Matching - https://github.com/ericrisco/rsc-harness/blob/main/skills/java/SKILL.md 


## Core Concept 4: Dynamic Method Dispatch

### Definitions

**Core Definition**: **Dynamic method dispatch** is the JVM mechanism that selects the most specific method implementation based on the runtime type of the object, not the compile-time type of the reference.

**Technical Definition**: When an overridden instance method is invoked, the Java Virtual Machine (JVM) calls the appropriate method for the object that is referred to in each variable. It does not call the method that is defined by the variable's type. This behavior is referred to as **virtual method invocation** and demonstrates an aspect of the important polymorphism features in the Java language . The compiler resolves the method **family** (signature) at compile time, but the JVM resolves the actual **implementation** at runtime .

**Beginner-Friendly Explanation**: Dynamic dispatch is like calling a company's main phone number and asking for "customer service." The phone system (JVM) routes your call to the right department (implementation) based on which company branch (object type) you're actually calling, not based on the number you dialed (reference type).

### Purposes

- To enable runtime polymorphism where subclasses provide specialized behavior.
- To allow code to be written against supertypes and work correctly with any subtype.
- To support the Open/Closed Principle (open for extension, closed for modification).
- To make systems extensible without modifying existing code.

### Syntax Rules and Structure

**Dynamic Dispatch Algorithm:**

1. **Compile-time**: The compiler verifies the method exists in the reference type and selects the method signature (method family) .
2. **Runtime**: The JVM looks at the actual class of the receiver object and finds the most specific implementation of that signature .
3. **Execution**: The found implementation is invoked.

**Syntax Rules:**

- Only instance methods participate in dynamic dispatch.
- `static`, `private`, and `final` methods use static binding.
- Fields are not polymorphic; they use the reference type .
- `super.method()` bypasses dynamic dispatch and calls the superclass version.

**Constraints and Limitations:**

- Dynamic dispatch has a slight performance overhead (vtable lookup).
- Cannot dynamically dispatch to a method not declared in the compile-time type.
- Type erasure can generate bridge methods that affect dispatch .

### Annotated Complete Code Examples

**Example 1: Dynamic Dispatch in Action**

```java
/**
 * Demonstrates dynamic method dispatch.
 */
class Payment {
    void process() {
        System.out.println("Processing payment generically");
    }
}

class CreditCardPayment extends Payment {
    @Override
    void process() {
        System.out.println("Processing credit card payment");
    }
}

class PayPalPayment extends Payment {
    @Override
    void process() {
        System.out.println("Processing PayPal payment");
    }
}

public class DispatchDemo {
    public static void main(String[] args) {
        Payment[] payments = {
            new Payment(),
            new CreditCardPayment(),
            new PayPalPayment()
        };

        for (Payment p : payments) {
            p.process(); // Dynamic dispatch selects the actual type's method
        }
    }
}
```

**Expected Output:**

```
Processing payment generically
Processing credit card payment
Processing PayPal payment
```

**Why This Output Occurs:**
- The array `Payment[]` holds objects of different runtime types.
- Each `p.process()` call dispatches to the implementation of the actual object's class .
- The compiler only knows `p` as `Payment`, but the JVM uses the runtime type.

### Real-World Cases

- **GUI frameworks**: `paint()` method overridden by different components.
- **Plugin architectures**: Plugins implement a common interface; dispatch calls the plugin's implementation.
- **Stream API**: `forEach()` dispatches to the consumer's `accept()` method.

### References

- Dev.java – Polymorphism (Virtual Method Invocation) - https://dev.java/learn/inheritance/polymorphism/ 
- University of Washington – Overloading vs. Overriding (Resolving Overriding) - https://courses.cs.washington.edu/courses/cse331/18au/lectures/lect06a-overloading.pdf 


## Core Concept 5: Safe Downcasting: `instanceof` and Pattern Matching

### Definitions

**Core Definition**: **Safe downcasting** uses the `instanceof` operator (and modern pattern matching) to verify an object's type before casting, preventing `ClassCastException` at runtime.

**Technical Definition**: The `instanceof` operator checks whether an object is an instance of a specific type (or its subtype). In modern Java (16+), **pattern matching for `instanceof`** combines the type check and cast into a single expression: `if (obj instanceof String s)` checks the type and assigns the cast value to `s` in one step . Java 21+ extends pattern matching to `switch` expressions, enabling exhaustive handling of sealed type hierarchies .

**Beginner-Friendly Explanation**: Safe downcasting is like checking the label on a package before opening it. If the label says "Circle," you open it as a circle. Modern pattern matching lets you say "if this is a Circle, call it circle and use it" in one clean step.

### Purposes

- To prevent `ClassCastException` by verifying types before casting.
- To enable safe type-specific operations in polymorphic code.
- To provide concise, readable type checking and casting.
- To enable exhaustive pattern matching over sealed hierarchies.

### Syntax Rules and Structure

**Traditional `instanceof` + Cast:**

```java
if (obj instanceof String) {
    String s = (String) obj;
    // use s
}
```

**Pattern Matching for `instanceof` (Java 16+):**

```java
if (obj instanceof String s) {
    // use s directly
}
```

**Pattern Matching in `switch` (Java 21+):**

```java
switch (obj) {
    case String s -> System.out.println(s.length());
    case Integer i -> System.out.println(i * 2);
    default -> System.out.println("Unknown");
}
```

**Component Breakdown:**

| Construct | Description |
|-----------|-------------|
| `instanceof Type` | Type check without binding. |
| `instanceof Type var` | Type check with pattern binding. |
| `case Type var ->` | Switch pattern with binding. |
| `when` guard | Additional condition on pattern . |

**Syntax Rules:**

- Pattern variables are in scope only when the pattern matches.
- Pattern matching in `switch` supports exhaustiveness checking for sealed types .
- Guards (`when`) add conditions to patterns .
- `instanceof` returns `false` for `null`; pattern matching follows the same rule.

**Constraints and Limitations:**

- Pattern matching cannot be used with generic types directly (due to erasure) .
- `switch` patterns require a default case unless the type is sealed and exhaustive .
- Pattern variables are implicitly final.

### Annotated Complete Code Examples

**Example 1: Traditional vs. Pattern Matching Downcast**

```java
/**
 * Demonstrates safe downcasting with instanceof and pattern matching.
 */
sealed interface Shape permits Circle, Square { }

record Circle(double radius) implements Shape { }
record Square(double side) implements Shape { }

public class SafeDowncastDemo {
    // Traditional approach
    static double areaTraditional(Shape shape) {
        if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return Math.PI * c.radius() * c.radius();
        } else if (shape instanceof Square) {
            Square s = (Square) shape;
            return s.side() * s.side();
        }
        return 0;
    }

    // Modern pattern matching approach
    static double areaModern(Shape shape) {
        return switch (shape) {
            case Circle c -> Math.PI * c.radius() * c.radius();
            case Square s -> s.side() * s.side();
            // No default needed: sealed hierarchy is exhaustive
        };
    }

    public static void main(String[] args) {
        Shape circle = new Circle(5);
        Shape square = new Square(4);

        System.out.println("Traditional circle area: " + areaTraditional(circle));
        System.out.println("Modern circle area: " + areaModern(circle));
        System.out.println("Modern square area: " + areaModern(square));
    }
}
```

**Expected Output:**

```
Traditional circle area: 78.53981633974483
Modern circle area: 78.53981633974483
Modern square area: 16.0
```

**Why This Output Occurs:**
- `areaTraditional` uses `instanceof` and explicit casts.
- `areaModern` uses pattern matching in `switch`, combining check and cast .
- The sealed interface ensures exhaustiveness; no `default` case is needed .

### Real-World Cases

- **Sealed hierarchies**: Pattern matching over `Result` types (Success/Failure).
- **Event processing**: Dispatching on event subtypes.
- **AST processing**: Pattern matching over expression nodes.

### References

- Modern Java in 2026 (CVUT) – Pattern Matching - https://cw.fel.cvut.cz/b252/_media/courses/b0b36pjv/prednasky/modern_java_2026.pdf 
- GitHub – Safe Downcasting with Pattern Matching - https://github.com/ericrisco/rsc-harness/blob/main/skills/java/SKILL.md 
- Dev.java – Type Erasure (bridge methods) - https://dev.java/learn/generics/type-erasure/ 


## Core Concept 6: Design Principles: Liskov Substitution Principle (LSP) and Coding to an Interface/Type

### Definitions

**Core Definition**: The **Liskov Substitution Principle (LSP)** states that objects of a superclass should be replaceable with objects of its subclasses without affecting the correctness of the program . **Coding to an interface** means declaring variables, parameters, and return types using interfaces or abstract supertypes rather than concrete implementations.

**Technical Definition**: LSP is one of the five SOLID principles. It requires that subtypes must be substitutable for their base types . This means subclasses cannot strengthen preconditions, weaken postconditions, violate invariants, or modify inherited state unexpectedly . Coding to an interface decouples client code from implementations, enabling polymorphism and making systems more flexible and testable.

**Beginner-Friendly Explanation**: LSP is like a universal power adapter. If your device works with a "Type A" plug, any "Type A" plug from any brand should work—if one doesn't, it violates the substitution principle. Coding to an interface is like saying "I need a plug that fits Type A" instead of "I need the specific brand X plug."

### Purposes

- To ensure polymorphic code works correctly with any subtype.
- To prevent subtle bugs from violated inheritance contracts.
- To decouple code from concrete implementations.
- To make systems more extensible, testable, and maintainable.

### Syntax Rules and Structure

**LSP Rules:**

| Rule | Meaning  |
|------|----------------------|
| Preconditions | Subclass cannot strengthen (require more). |
| Postconditions | Subclass cannot weaken (promise less). |
| Invariants | Subclass must maintain parent's invariants. |
| History | Subclass cannot modify inherited state unexpectedly. |

**Coding to an Interface:**

```java
// BAD: Depends on concrete class
ArrayList<String> list = new ArrayList<>();

// GOOD: Depends on interface
List<String> list = new ArrayList<>();
```

**Syntax Rules:**

- LSP violations can be detected by `instanceof` checks before method calls .
- Subclasses should not throw exceptions the parent doesn't throw .
- Use interfaces or abstract classes for polymorphic types.
- Return types should be interfaces where possible.

**Constraints and Limitations:**

- LSP is a semantic constraint, not enforced by the compiler.
- Violating LSP leads to fragile code that breaks when subtypes are substituted.
- Coding to interfaces can introduce indirection but improves flexibility.

### Annotated Complete Code Examples

**Example 1: LSP Violation and Fix**

```java
/**
 * Demonstrates LSP violation and correction.
 */
// VIOLATION: Square changes Rectangle's behavior
class Rectangle {
    protected int width, height;
    public void setWidth(int w) { width = w; }
    public void setHeight(int h) { height = h; }
    public int area() { return width * height; }
}

class SquareViolation extends Rectangle {
    @Override
    public void setWidth(int w) { width = w; height = w; }
    @Override
    public void setHeight(int h) { width = h; height = h; }
}

// FIX: Separate abstractions with common interface
interface Shape {
    int area();
}

class RectangleFixed implements Shape {
    private final int width, height;
    RectangleFixed(int w, int h) { width = w; height = h; }
    public int area() { return width * height; }
}

class SquareFixed implements Shape {
    private final int side;
    SquareFixed(int s) { side = s; }
    public int area() { return side * side; }
}

public class LspDemo {
    static void testRectangle(Rectangle r) {
        r.setWidth(5);
        r.setHeight(4);
        System.out.println("Expected area: 20, Actual: " + r.area());
    }

    public static void main(String[] args) {
        System.out.println("Violation:");
        testRectangle(new SquareViolation()); // Breaks LSP

        System.out.println("\nFixed:");
        System.out.println("Rectangle area: " + new RectangleFixed(5, 4).area());
        System.out.println("Square area: " + new SquareFixed(4).area());
    }
}
```

**Expected Output:**

```
Violation:
Expected area: 20, Actual: 16

Fixed:
Rectangle area: 20
Square area: 16
```

**Why This Output Occurs:**
- `SquareViolation` changes `setWidth` to also set height, violating the Rectangle contract .
- `testRectangle` expects area 20 but gets 16.
- The fixed version separates `Rectangle` and `Square` as independent `Shape` implementations, preserving LSP.

### Real-World Cases

- **Collections**: `List` interface allows `ArrayList` or `LinkedList` without client changes.
- **Payment processing**: `PaymentMethod` interface with `CreditCard`, `PayPal` implementations.
- **Plugin systems**: Coding to interfaces enables dynamic plugin loading.

### References

- GitHub – SOLID Principles (LSP) - https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/decebals/claude-code-java/solid-principles/SKILL.md 
- GitHub – SOLID Principles in Java (LSP) - https://raw.githubusercontent.com/MultiChatNetwork/engineering-multichat/765e5d79ff24b2079453f7df69cef9caa24f4bf9/posts/java-solid-principles.md 
- Baeldung – Liskov Substitution Principle - https://www.baeldung.com/java-liskov-substitution-principle 


## References

- Dev.java – Polymorphism - https://dev.java/learn/inheritance/polymorphism/ 
- University of Washington – Overloading vs. Overriding - https://courses.cs.washington.edu/courses/cse331/18au/lectures/lect06a-overloading.pdf 
- Modern Java in 2026 (CVUT) – Pattern Matching - https://cw.fel.cvut.cz/b252/_media/courses/b0b36pjv/prednasky/modern_java_2026.pdf 
- GitHub – Safe Downcasting with Pattern Matching - https://github.com/ericrisco/rsc-harness/blob/main/skills/java/SKILL.md 
- GitHub – SOLID Principles (LSP) - https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/decebals/claude-code-java/solid-principles/SKILL.md 
- GitHub – SOLID Principles in Java (LSP) - https://raw.githubusercontent.com/MultiChatNetwork/engineering-multichat/765e5d79ff24b2079453f7df69cef9caa24f4bf9/posts/java-solid-principles.md 
- Baeldung – Liskov Substitution Principle - https://www.baeldung.com/java-liskov-substitution-principle 
- Dev.java – Type Erasure - https://dev.java/learn/generics/type-erasure/ 
- Oracle – Wildcards Guidelines - https://docs.oracle.com/javase/tutorial/java/generics/wildcardGuidelines.html 