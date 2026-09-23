# Java Inheritance: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Inheritance** is a fundamental object-oriented programming mechanism in Java that allows a class (the **subclass** or **child class**) to acquire the fields and methods of another class (the **superclass** or **parent class**). It establishes an **"Is-A" relationship** between classes, enabling code reuse and hierarchical classification.

### Technical Definition

Inheritance in Java is implemented through the `extends` keyword for classes and `implements` for interfaces. A subclass inherits all non-private members of its superclass and can add new members, override inherited methods, or hide inherited fields. Java supports single inheritance for classes (a class can extend only one superclass) but multiple inheritance for interfaces. Every class in Java implicitly extends `java.lang.Object` unless it explicitly extends another class. Constructors are not inherited; instead, subclass constructors must invoke a superclass constructor, either implicitly via `super()` or explicitly. The `final` keyword prevents inheritance or method overriding, while `sealed` classes (introduced in Java 17) allow controlled inheritance by specifying which classes may extend them.

### Beginner-Friendly Explanation

Inheritance is like a family tree. A child inherits traits from their parents—eye color, height, maybe a talent for music. In Java, a "child" class inherits fields and methods from its "parent" class. The child can also have its own unique traits (new fields and methods) and can even change inherited behaviors (override methods). This lets you write the common code once in the parent and reuse it in all the children, while still allowing each child to be special.

### Key Characteristics

- **Is-A relationship**: A subclass is a specialized version of its superclass.
- **Code reuse**: Common fields and methods are defined once in the superclass.
- **Single inheritance for classes**: A class can extend exactly one superclass.
- **Multiple inheritance for interfaces**: A class can implement multiple interfaces.
- **Constructor chaining**: Subclass constructors invoke superclass constructors first.
- **`super` keyword**: Accesses superclass members (fields, methods, constructors).
- **`final` restriction**: Prevents inheritance or overriding.
- **`sealed` restriction**: Controls which classes may extend a type.

### Prerequisites

- Basic Java syntax (classes, fields, methods, constructors).
- Understanding of access modifiers (`private`, `protected`, `public`, package-private).
- Familiarity with the `this` keyword and object creation.

### Related Programming Areas

- **Polymorphism**: Subclasses can be treated as superclass types.
- **Abstract Classes and Interfaces**: Contracts for inheritance hierarchies.
- **Design Patterns**: Template Method, Strategy, and Factory patterns rely on inheritance.
- **Domain Modeling**: Representing real-world hierarchies (e.g., `Animal` → `Dog`, `Cat`).

### Core Concepts / Features

1. The `extends` Keyword
2. Parent Class (Superclass) and Child Class (Subclass)
3. Reuse Through Inheritance (Is-A Relationship)
4. Constructor Behavior and Initialization Order
5. The `super` Keyword
6. Preventing Inheritance with `final`
7. Sealed Classes and Interfaces


## Core Concept 1: The `extends` Keyword

### Definitions

**Core Definition**: The **`extends`** keyword is used in a class declaration to indicate that the class inherits from another class (its superclass).

**Technical Definition**: In Java, the `extends` clause in a class declaration specifies the direct superclass of the class being declared. A class inherits all members (fields, methods, and nested classes) of its superclass that are accessible (not `private`). The subclass can then add new members, override inherited methods, or hide inherited fields. If a class declaration has no `extends` clause, the class implicitly extends `java.lang.Object`.

**Beginner-Friendly Explanation**: The `extends` keyword is like saying "this class is a more specific version of that class." For example, `class Dog extends Animal` means "a Dog is a kind of Animal." The Dog automatically gets all the general Animal traits, and can add its own dog-specific traits.

### Purposes

- To establish an inheritance relationship between two classes.
- To reuse fields and methods from the superclass.
- To create a hierarchical classification of types.
- To enable polymorphism through superclass references.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[access_modifier] [modifiers] class SubclassName extends SuperclassName {
    // subclass body
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[access_modifier]` | `public`, `protected`, or package-private. |
| `[modifiers]` | `abstract`, `final`, `sealed`, `non-sealed`, etc. |
| `class` | Keyword declaring a class. |
| `SubclassName` | The name of the subclass. |
| `extends` | Keyword indicating inheritance. |
| `SuperclassName` | The name of the superclass (only one). |

**Syntax Rules:**

- A class can extend **exactly one** superclass.
- If no `extends` clause is present, the class extends `Object`.
- The superclass must be accessible (not `private` or in an inaccessible package).
- The superclass cannot be `final`.
- The superclass cannot be `sealed` unless the subclass is listed in its `permits` clause.

**Constraints and Limitations:**

- Cannot extend a `final` class.
- Cannot extend a `sealed` class unless explicitly permitted.
- Cannot extend a class with no accessible no-argument constructor without explicit `super(...)` call.
- Interfaces use `extends` to inherit other interfaces, but classes use `implements` for interfaces.

### Annotated Complete Code Examples

**Example 1: Basic Inheritance with `extends`**

```java
/**
 * Demonstrates the extends keyword.
 */
class Animal {
    void eat() {
        System.out.println("This animal eats food.");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("The dog barks.");
    }
}

public class ExtendsDemo {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.eat();  // Inherited from Animal
        dog.bark(); // Defined in Dog
    }
}
```

**Expected Output:**

```
This animal eats food.
The dog barks.
```

**Why This Output Occurs:**
- `Dog extends Animal` means `Dog` inherits `eat()`.
- `dog.eat()` calls the inherited method.
- `dog.bark()` calls the method defined in `Dog`.

### Real-World Cases

- **`class Student extends Person`**: Student is a kind of Person.
- **`class SavingsAccount extends Account`**: Savings account is a kind of account.
- **`class IOException extends Exception`**: I/O exception is a kind of exception.

### References

- Oracle Java Tutorials – Inheritance - https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html
- Java Language Specification – Class Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html


## Core Concept 2: Parent Class (Superclass) and Child Class (Subclass)

### Definitions

**Core Definition**: A **superclass** (parent class) is the class being inherited from; a **subclass** (child class) is the class that inherits from the superclass.

**Technical Definition**: A subclass inherits all `public` and `protected` members of its superclass, as well as package-private members if the subclass is in the same package. `private` members are not accessible by the subclass but exist in the object. The subclass can access inherited members directly (by name) or via `super`. The subclass can also add new fields and methods, and can override inherited methods to provide specialized behavior.

**Beginner-Friendly Explanation**: The superclass is the "parent" with general characteristics; the subclass is the "child" with more specific characteristics. A child inherits the parent's traits and can develop their own unique traits.

### Purposes

- To model "Is-A" relationships in domain hierarchies.
- To share common code among related classes.
- To allow specialized classes to extend and customize general behavior.
- To support polymorphism (treating subclasses as superclass types).

### Syntax Rules and Structure

**Complete General Syntax:**

```java
class Superclass {
    // common members
}

class Subclass extends Superclass {
    // inherited members + new members
}
```

**Access to Inherited Members:**

| Member Type | Accessible in Subclass? |
|-------------|------------------------|
| `public` | Yes |
| `protected` | Yes |
| package-private (same package) | Yes |
| package-private (different package) | No |
| `private` | No (exists but inaccessible) |

**Syntax Rules:**

- The subclass is a subtype of the superclass (Liskov Substitution Principle).
- A superclass reference can refer to a subclass object.
- A subclass reference cannot refer to a superclass object without a cast.
- The subclass can override inherited methods (unless they are `final`).

**Constraints and Limitations:**

- Private members are not inherited (not accessible).
- Constructors are not inherited.
- Static members are inherited but belong to the class, not instances.
- Overriding methods must have compatible return types (covariant returns allowed).

### Annotated Complete Code Examples

**Example 1: Superclass and Subclass**

```java
/**
 * Demonstrates superclass and subclass relationships.
 */
class Vehicle {
    protected String brand = "Generic";

    void honk() {
        System.out.println("Beep beep!");
    }
}

class Car extends Vehicle {
    private String model = "Sedan";

    void display() {
        System.out.println("Brand: " + brand); // Inherited protected field
        System.out.println("Model: " + model); // Own field
    }
}

public class SuperSubDemo {
    public static void main(String[] args) {
        Car myCar = new Car();
        myCar.honk();    // Inherited method
        myCar.display(); // Own method

        // Polymorphism: superclass reference to subclass object
        Vehicle v = new Car();
        v.honk(); // Calls inherited method
        // v.display(); // Compile error: display() not in Vehicle
    }
}
```

**Expected Output:**

```
Beep beep!
Brand: Generic
Model: Sedan
Beep beep!
```

**Why This Output Occurs:**
- `Car` inherits `brand` and `honk()` from `Vehicle`.
- `myCar.honk()` calls the inherited method.
- `myCar.display()` accesses both the inherited `brand` and its own `model`.
- `v.honk()` uses a `Vehicle` reference but calls the `Car` object's inherited `honk()`.

### Real-World Cases

- **`class Manager extends Employee`**: Manager is a specialized Employee.
- **`class Circle extends Shape`**: Circle is a specialized Shape.
- **`class ArrayList extends AbstractList`**: ArrayList is a specialized list.

### References

- Oracle Java Tutorials – Inheritance - https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html
- Cornell CS 2110 – Inheritance - https://www.cs.cornell.edu/courses/cs2110/2026sp/lectures/


## Core Concept 3: Reuse Through Inheritance (Is-A Relationship)

### Definitions

**Core Definition**: **Reuse through inheritance** is the practice of defining common fields and methods in a superclass so that subclasses can inherit them without reimplementation. The **"Is-A" relationship** means a subclass is a specialized kind of its superclass.

**Technical Definition**: Inheritance promotes code reuse by allowing subclasses to inherit the state and behavior of their superclass. The "Is-A" relationship is a semantic constraint: a subclass must be substitutable for its superclass (Liskov Substitution Principle). This means a subclass should not weaken the superclass's contract (e.g., by throwing new exceptions or violating invariants).

**Beginner-Friendly Explanation**: Inheritance is like a recipe book. The basic recipe (superclass) has common steps. A specialized recipe (subclass) says "use the basic recipe, but add these extra ingredients." You don't have to rewrite the basic steps—you just reuse them.

### Purposes

- To eliminate duplicate code by centralizing common functionality.
- To model real-world hierarchies accurately.
- To enable polymorphism and dynamic method dispatch.
- To make code more maintainable (changes in the superclass propagate to subclasses).

### Syntax Rules and Structure

**Is-A Relationship Examples:**

| Subclass | Superclass | Is-A? |
|----------|------------|-------|
| `Dog` | `Animal` | A Dog **is an** Animal ✓ |
| `Car` | `Vehicle` | A Car **is a** Vehicle ✓ |
| `Manager` | `Employee` | A Manager **is an** Employee ✓ |
| `Square` | `Rectangle` | A Square **is a** Rectangle ✓ (mathematically) |

**Syntax Rules:**

- Use inheritance only when the "Is-A" relationship is true.
- The subclass should be substitutable for the superclass.
- Avoid inheritance for mere code reuse when the "Is-A" relationship doesn't hold (prefer composition).

**Constraints and Limitations:**

- Inheritance creates tight coupling between superclass and subclass.
- Changes to the superclass can break subclasses (fragile base class problem).
- Overuse of inheritance can lead to deep, complex hierarchies.

### Annotated Complete Code Examples

**Example 1: Code Reuse Through Inheritance**

```java
/**
 * Demonstrates code reuse through inheritance.
 */
class Employee {
    protected String name;
    protected double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public void work() {
        System.out.println(name + " is working.");
    }

    public double getSalary() {
        return salary;
    }
}

class Manager extends Employee {
    private double bonus;

    public Manager(String name, double salary, double bonus) {
        super(name, salary); // Reuse Employee constructor
        this.bonus = bonus;
    }

    @Override
    public void work() {
        System.out.println(name + " is managing the team.");
    }

    public double getTotalCompensation() {
        return getSalary() + bonus; // Reuse getSalary()
    }
}

public class ReuseDemo {
    public static void main(String[] args) {
        Manager mgr = new Manager("Alice", 80000, 20000);
        mgr.work();
        System.out.println("Total compensation: " + mgr.getTotalCompensation());
    }
}
```

**Expected Output:**

```
Alice is managing the team.
Total compensation: 100000.0
```

**Why This Output Occurs:**
- `Manager` inherits `name`, `salary`, and `getSalary()` from `Employee`.
- The `Manager` constructor calls `super(name, salary)` to reuse the `Employee` constructor.
- `Manager` overrides `work()` to provide specialized behavior.
- `getTotalCompensation()` reuses `getSalary()` from the superclass.

### Real-World Cases

- **`ArrayList extends AbstractList`**: Reuses common list operations.
- **`HttpServlet extends GenericServlet`**: Reuses servlet lifecycle methods.
- **`JFrame extends Frame`**: Reuses window management code.

### References

- Oracle Java Tutorials – Inheritance - https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html
- Cornell CS 2110 – Inheritance and Code Reuse - https://www.cs.cornell.edu/courses/cs2110/


## Core Concept 4: Constructor Behavior and Initialization Order

### Definitions

**Core Definition**: **Constructor behavior in inheritance** refers to how constructors of superclasses and subclasses are invoked when an object is created. **Initialization order** is the sequence in which fields and constructor bodies execute.

**Technical Definition**: When a subclass object is created, the superclass constructor must be invoked before the subclass constructor body executes. If the subclass constructor does not explicitly call `super(...)` or `this(...)`, the compiler inserts an implicit `super()` call. The initialization order is: (1) memory allocated, fields set to defaults; (2) superclass constructor executes (including its field initializers and instance blocks); (3) subclass field initializers and instance blocks execute; (4) subclass constructor body executes.

**Beginner-Friendly Explanation**: Building a house (object) starts with the foundation (superclass). You can't paint the walls (subclass) before the foundation is laid. Java automatically ensures the superclass is fully initialized before the subclass continues.

### Purposes

- To ensure superclass state is initialized before subclass state.
- To allow subclasses to pass required data to superclass constructors.
- To understand the sequence of field initializers, instance blocks, and constructor bodies.
- To avoid accessing uninitialized subclass fields from superclass constructors.

### Syntax Rules and Structure

**Initialization Order:**

1. **Allocate memory**: Fields set to default values (`0`, `false`, `null`).
2. **Invoke superclass constructor**: Recursively, from the top of the hierarchy down.
3. **Execute superclass field initializers and instance blocks**: In textual order.
4. **Execute superclass constructor body**.
5. **Execute subclass field initializers and instance blocks**: In textual order.
6. **Execute subclass constructor body**.

**Syntax Rules:**

- The first statement of any constructor must be `this(...)`, `super(...)`, or an implicit `super()`.
- If the superclass lacks a no-argument constructor, the subclass must explicitly call `super(...)`.
- Field initializers and instance blocks run **after** the superclass constructor but **before** the subclass constructor body.
- Calling overridable methods from a constructor can access uninitialized subclass fields.

**Constraints and Limitations:**

- Cannot access subclass-specific fields in a superclass constructor (they are not initialized yet).
- Overridable methods called from constructors can cause subtle bugs.
- The implicit `super()` fails if the superclass has no accessible no-argument constructor.

### Annotated Complete Code Examples

**Example 1: Initialization Order in Inheritance**

```java
/**
 * Demonstrates initialization order in inheritance.
 */
class Parent {
    // Field initializer
    protected int parentField = initParentField();

    // Instance initializer block
    {
        System.out.println("Parent instance block");
    }

    // Constructor
    public Parent() {
        System.out.println("Parent constructor body");
    }

    private int initParentField() {
        System.out.println("Parent field initializer");
        return 10;
    }
}

class Child extends Parent {
    // Field initializer
    private int childField = initChildField();

    // Instance initializer block
    {
        System.out.println("Child instance block");
    }

    // Constructor
    public Child() {
        // Implicit super() call here
        System.out.println("Child constructor body");
    }

    private int initChildField() {
        System.out.println("Child field initializer");
        return 20;
    }
}

public class InitOrderDemo {
    public static void main(String[] args) {
        System.out.println("Creating Child:");
        new Child();
    }
}
```

**Expected Output:**

```
Creating Child:
Parent field initializer
Parent instance block
Parent constructor body
Child field initializer
Child instance block
Child constructor body
```

**Why This Output Occurs:**
- `new Child()` triggers `Child` constructor.
- Implicit `super()` invokes `Parent` constructor.
- `Parent` field initializer runs first, then `Parent` instance block, then `Parent` constructor body.
- After `super()` returns, `Child` field initializer runs, then `Child` instance block, then `Child` constructor body.

### Real-World Cases

- **Framework classes**: Understanding initialization order helps debug framework integration.
- **Abstract classes**: Subclasses must call `super(...)` to initialize abstract class state.
- **Immutable classes**: All fields must be initialized in constructors.

### References

- Oracle Java Tutorials – Inheritance (Constructors) - https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html
- Cornell CS 2110 – Constructors and Initialization Order - https://www.cs.cornell.edu/courses/cs2110/


## Core Concept 5: The `super` Keyword

### Definitions

**Core Definition**: The **`super`** keyword is a reference to the superclass of the current object, used to access superclass members (constructors, methods, fields).

**Technical Definition**: `super` can be used in three ways: (1) `super(...)` invokes a superclass constructor and must be the first statement in a subclass constructor; (2) `super.methodName(...)` invokes a superclass method that has been overridden; (3) `super.fieldName` accesses a superclass field that has been hidden by a subclass field.

**Beginner-Friendly Explanation**: `super` is like saying "the parent version of this." If you've overridden a method, `super.method()` calls the original parent version. If you have a field with the same name as a parent field, `super.field` accesses the parent's field.

### Purposes

- To invoke a superclass constructor from a subclass constructor.
- To call an overridden superclass method from a subclass method.
- To access a hidden superclass field.
- To reuse superclass initialization and behavior.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
super(...)           // Call superclass constructor
super.methodName()   // Call superclass method
super.fieldName      // Access superclass field
```

**Component Breakdown:**

| Usage | Description |
|-------|-------------|
| `super(args)` | Invokes a superclass constructor; must be first statement. |
| `super.method()` | Invokes the superclass's version of an overridden method. |
| `super.field` | Accesses a superclass field hidden by a subclass field. |

**Syntax Rules:**

- `super(...)` must be the first statement in a constructor.
- `super.method()` can be used anywhere in an instance method.
- `super.field` can be used to access hidden fields.
- `super` cannot be used in a static context.
- `super` cannot be used in a constructor before `this(...)` or `super(...)`.

**Constraints and Limitations:**

- Cannot use `super` in static methods.
- Cannot call `super.super.method()` (no grandparent access directly).
- `super(...)` fails if the superclass has no matching constructor.

### Annotated Complete Code Examples

**Example 1: Using `super` for Constructors and Methods**

```java
/**
 * Demonstrates all uses of super.
 */
class Base {
    protected String message = "Base message";

    public Base() {
        System.out.println("Base constructor");
    }

    public Base(String msg) {
        this.message = msg;
        System.out.println("Base parameterized: " + msg);
    }

    public void display() {
        System.out.println("Base display: " + message);
    }
}

class Derived extends Base {
    protected String message = "Derived message";

    public Derived() {
        super("Hello from Derived"); // super constructor call
        System.out.println("Derived constructor");
    }

    public void display() {
        super.display(); // Call superclass method
        System.out.println("Derived display: " + message);
        System.out.println("Super message: " + super.message); // Access hidden field
    }
}

public class SuperDemo {
    public static void main(String[] args) {
        Derived d = new Derived();
        d.display();
    }
}
```

**Expected Output:**

```
Base parameterized: Hello from Derived
Derived constructor
Base display: Hello from Derived
Derived display: Derived message
Super message: Hello from Derived
```

**Why This Output Occurs:**
- `super("Hello from Derived")` calls the parameterized `Base` constructor.
- `super.display()` calls `Base`'s `display()` method.
- `super.message` accesses `Base`'s `message` field, hiding `Derived`'s `message`.

### Real-World Cases

- **Overridden methods**: Calling `super.toString()` before adding subclass details.
- **Constructor chaining**: Passing required data to superclass constructors.
- **Framework integration**: Extending framework classes and calling `super` methods.

### References

- Oracle Java Tutorials – Inheritance (super) - https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html
- Java Language Specification – super - https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html


## Core Concept 6: Preventing Inheritance with `final`

### Definitions

**Core Definition**: The **`final`** keyword prevents a class from being extended or a method from being overridden.

**Technical Definition**: A `final` class cannot be subclassed. A `final` method cannot be overridden by subclasses. `final` is used to ensure that a class's behavior cannot be changed through inheritance, which is particularly useful for immutable classes (like `String`) and critical methods whose behavior must remain consistent.

**Beginner-Friendly Explanation**: Marking a class `final` is like saying "this is the last version—no more changes." Marking a method `final` is like saying "this specific behavior cannot be changed by children."

### Purposes

- To prevent unwanted subclassing.
- To protect critical methods from being overridden.
- To ensure consistency and security of immutable classes.
- To allow JVM optimizations (e.g., inlining) by knowing methods won't be overridden.

### Syntax Rules and Structure

**Complete General Syntax (Final Class):**

```java
public final class ClassName {
    // cannot be extended
}
```

**Complete General Syntax (Final Method):**

```java
public final ReturnType methodName() {
    // cannot be overridden
}
```

**Component Breakdown:**

| Usage | Effect |
|-------|--------|
| `final class` | Class cannot be extended. |
| `final method` | Method cannot be overridden. |
| `final field` | Field cannot be reassigned (constant). |

**Syntax Rules:**

- `final` can be applied to classes, methods, fields, and parameters.
- A `final` class cannot be extended.
- A `final` method cannot be overridden.
- `final` and `abstract` cannot be used together.

**Constraints and Limitations:**

- `final` classes cannot be extended, limiting flexibility.
- `final` methods cannot be overridden, which may hinder testing (mocking).
- Use `final` judiciously; overuse can make code rigid.

### Annotated Complete Code Examples

**Example 1: Final Class and Method**

```java
/**
 * Demonstrates final class and final method.
 */
final class ImmutablePoint {
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public final int getX() { return x; }
    public final int getY() { return y; }
}

// class ExtendedPoint extends ImmutablePoint { } // Compile error: cannot extend final class

class Base {
    public final void criticalMethod() {
        System.out.println("Critical behavior locked");
    }
}

class Derived extends Base {
    // public void criticalMethod() { } // Compile error: cannot override final method
}

public class FinalDemo {
    public static void main(String[] args) {
        ImmutablePoint p = new ImmutablePoint(3, 4);
        System.out.println("Point: (" + p.getX() + ", " + p.getY() + ")");

        Derived d = new Derived();
        d.criticalMethod();
    }
}
```

**Expected Output:**

```
Point: (3, 4)
Critical behavior locked
```

**Why This Output Occurs:**
- `ImmutablePoint` is `final`, so it cannot be extended.
- `getX()` and `getY()` are `final`, so they cannot be overridden.
- `criticalMethod()` is `final`, so `Derived` cannot override it.

### Real-World Cases

- **`String` class**: `final` to ensure immutability and thread safety.
- **`Integer`, `Double`**: Wrapper classes are `final`.
- **Critical algorithms**: `final` methods in frameworks to prevent unintended behavior changes.

### References

- IIT Bombay – Writing Final Classes and Methods - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/java/javaOO/final.html
- Buyya – Final and Abstract Classes - http://buyya.com/254/Lectures/Lecture13.pdf
- GitHub – Java Final Keyword - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/language/java/4-java-object-oriented-programming/11-java-final-keyword.mdx


## Core Concept 7: Sealed Classes and Interfaces

### Definitions

**Core Definition**: **Sealed classes and interfaces** restrict which other classes or interfaces may extend or implement them, allowing the developer to control the inheritance hierarchy explicitly.

**Technical Definition**: A sealed class or interface is declared with the `sealed` modifier and a `permits` clause that lists the classes or interfaces allowed to extend or implement it. Every permitted subclass must be declared either `final` (cannot be extended further), `sealed` (continues the sealed hierarchy with its own permits clause), or `non-sealed` (reopens the hierarchy for unrestricted extension). Permitted subclasses must be in the same module (or same package for unnamed modules) as the sealed class.

**Beginner-Friendly Explanation**: A sealed class is like a VIP club with a guest list. Only the classes on the list (the `permits` clause) are allowed in (can extend the sealed class). Each guest must then decide: stay exclusive (`sealed`), close the door behind them (`final`), or open the club to everyone (`non-sealed`).

### Purposes

- To model domains where the set of possible subtypes is known and fixed.
- To enable exhaustive pattern matching (the compiler knows all permitted subclasses).
- To prevent unwanted or unknown subclasses in APIs.
- To provide a middle ground between `final` (no extension) and open inheritance (unrestricted extension).

### Syntax Rules and Structure

**Complete General Syntax (Sealed Class):**

```java
public sealed class Shape permits Circle, Square, Rectangle {
    // class body
}
```

**Complete General Syntax (Permitted Subclasses):**

```java
public final class Circle extends Shape { }       // Cannot be extended
public non-sealed class Square extends Shape { }  // Reopens for extension
public sealed class Rectangle extends Shape permits FilledRectangle { }
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `sealed` | Modifier indicating restricted inheritance. |
| `permits` | Clause listing allowed subclasses. |
| `final` | Permitted subclass cannot be extended further. |
| `sealed` | Permitted subclass continues the sealed hierarchy. |
| `non-sealed` | Permitted subclass reopens for unrestricted extension. |

**Syntax Rules:**

- A sealed class must have a `permits` clause unless all permitted subclasses are in the same file (then it can be omitted).
- Every permitted subclass must be `final`, `sealed`, or `non-sealed`.
- Permitted subclasses must directly extend/implement the sealed type.
- In named modules, permitted subclasses must be in the same module.
- In unnamed modules, permitted subclasses must be in the same package.

**Constraints and Limitations:**

- Sealed hierarchies are not intended to span maintenance domains.
- Changing a non-sealed class to sealed can break existing binaries.
- A sealed class cannot be instantiated if it is abstract.

### Annotated Complete Code Examples

**Example 1: Sealed Class Hierarchy**

```java
/**
 * Demonstrates sealed classes and interfaces.
 */
public sealed interface Shape permits Circle, Square, Rectangle {
    double area();
}

// final: cannot be extended further
final class Circle implements Shape {
    private final double radius;
    Circle(double radius) { this.radius = radius; }
    public double area() { return Math.PI * radius * radius; }
}

// non-sealed: reopens for unrestricted extension
non-sealed class Square implements Shape {
    private final double side;
    Square(double side) { this.side = side; }
    public double area() { return side * side; }
}

// sealed: continues the sealed hierarchy
sealed class Rectangle implements Shape permits FilledRectangle {
    protected final double width, height;
    Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    public double area() { return width * height; }
}

// final permitted subclass of Rectangle
final class FilledRectangle extends Rectangle {
    private final String color;
    FilledRectangle(double width, double height, String color) {
        super(width, height);
        this.color = color;
    }
}

// Extended from non-sealed Square
class ColoredSquare extends Square {
    private final String color;
    ColoredSquare(double side, String color) {
        super(side);
        this.color = color;
    }
}

public class SealedDemo {
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle(5),
            new Square(4),
            new FilledRectangle(3, 4, "red"),
            new ColoredSquare(6, "blue")
        };

        for (Shape s : shapes) {
            System.out.printf("%s area: %.2f%n",
                s.getClass().getSimpleName(), s.area());
        }
    }
}
```

**Expected Output:**

```
Circle area: 78.54
Square area: 16.00
FilledRectangle area: 12.00
ColoredSquare area: 36.00
```

**Why This Output Occurs:**
- `Shape` is sealed, permitting only `Circle`, `Square`, and `Rectangle`.
- `Circle` is `final` — cannot be extended.
- `Square` is `non-sealed` — `ColoredSquare` can extend it.
- `Rectangle` is `sealed` — only `FilledRectangle` can extend it.
- `FilledRectangle` is `final` — cannot be extended.

### Real-World Cases

- **Shape hierarchies**: Sealed `Shape` with permitted `Circle`, `Square`, `Rectangle`.
- **Expression trees**: Sealed `Expr` with `Constant`, `Add`, `Multiply`.
- **Payment methods**: Sealed `Payment` with `CreditCard`, `PayPal`, `Crypto`.
- **Pattern matching**: Sealed classes enable exhaustive `switch` expressions.

### References

- Oracle Java SE 23 – Sealed Classes and Interfaces - https://docs.oracle.com/javase/jp/23/language/sealed-classes-and-interfaces.html
- OpenJDK – JEP 409: Sealed Classes - https://openjdk.org/jeps/409
- Stack Overflow – Difference between final and non-sealed - https://stackoverflow.com/questions/63972130/what-is-the-difference-between-a-final-and-a-non-sealed-class-in-java-15s-seale
- OpenJDK – JDK-8341669: Access to private classes in permits clause - https://bugs.openjdk.org/browse/JDK-8341669


## References

- Oracle Java Tutorials – Inheritance - https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html
- Oracle Java SE 23 – Sealed Classes and Interfaces - https://docs.oracle.com/javase/jp/23/language/sealed-classes-and-interfaces.html
- OpenJDK – JEP 409: Sealed Classes - https://openjdk.org/jeps/409
- IIT Bombay – Writing Final Classes and Methods - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/java/javaOO/final.html
- Buyya – Final and Abstract Classes - http://buyya.com/254/Lectures/Lecture13.pdf
- GitHub – Java Final Keyword - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/language/java/4-java-object-oriented-programming/11-java-final-keyword.mdx
- Stack Overflow – Difference between final and non-sealed - https://stackoverflow.com/questions/63972130/what-is-the-difference-between-a-final-and-a-non-sealed-class-in-java-15s-seale
- OpenJDK – JDK-8341669: Access to private classes in permits clause - https://bugs.openjdk.org/browse/JDK-8341669
- Stack Overflow – Sealed classes implementation patterns - https://stackoverflow.com/revisions/464af06b-bc34-4f38-8594-7b2bba52e42c/view-source
- Java Language Specification – Class Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html