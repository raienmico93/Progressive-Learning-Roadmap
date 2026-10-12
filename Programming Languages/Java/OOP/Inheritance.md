# Java Inheritance: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Inheritance is a fundamental mechanism in object-oriented programming where a new class (subclass) acquires the properties and behaviours of an existing class (superclass), enabling code reuse and the creation of hierarchical relationships between classes.

**Technical Definition:** In the Java language, classes can be derived from other classes, thereby inheriting fields and methods from those classes. A class that is derived from another class is called a subclass (also a derived class, extended class, or child class). The class from which the subclass is derived is called a superclass (also a base class or a parent class). Excepting `Object`, which has no superclass, every class has one and only one direct superclass (single inheritance).

**Beginner-Friendly Explanation:** Inheritance is like a family tree for classes. Just as you inherit traits from your parents (like eye colour or height), a Java class can inherit fields and methods from another class. This means you don't have to write the same code again — you can create a new class that automatically has all the features of an existing class, and then add or modify things as needed.

### Key Characteristics

- **Single Inheritance:** Java supports single inheritance for classes, meaning a class can extend only one superclass.
- **Code Reuse:** Subclasses inherit all members (fields, methods, and nested classes) from their superclass, reducing code duplication.
- **`is-a` Relationship:** Inheritance models an "is-a" relationship (e.g., a `MountainBike` is-a `Bicycle`).
- **`Object` as Root:** All classes in Java ultimately inherit from `java.lang.Object`.
- **Constructors Are Not Inherited:** Constructors are not members, so they are not inherited by subclasses, but the constructor of the superclass can be invoked from the subclass.
- **Polymorphism:** Inheritance enables polymorphic behaviour where a subclass object can be treated as an object of its superclass.

### Prerequisites

- Basic understanding of Java classes and objects
- Familiarity with fields, methods, and constructors
- Knowledge of access modifiers (`private`, `protected`, `public`)
- Awareness of the `this` keyword and object creation

### Related Programming Areas

- **Polymorphism:** Inheritance is the foundation for runtime polymorphism through method overriding.
- **Abstract Classes and Interfaces:** These are advanced forms of inheritance used for defining contracts and partial implementations.
- **Design Patterns:** The Template Method pattern relies heavily on inheritance; the Strategy pattern favours composition.
- **API Design:** Inheritance hierarchies are common in frameworks (e.g., `InputStream` hierarchy in Java I/O).
- **Refactoring:** Inheritance is often refactored into composition to reduce coupling.

### Core Concepts / Features

---

## 1. Inheritance

### Definitions

**Core Definition:** Inheritance is the mechanism by which one class acquires the properties and behaviours of another class.

**Technical Definition:** Inheritance allows a class (subclass) to inherit fields and methods from another class (superclass) using the `extends` keyword. A subclass inherits all the members from its superclass, including fields, methods, and nested classes. The subclass can then add new members or override inherited methods to modify behaviour.

**Beginner-Friendly Explanation:** Inheritance is like getting a starter kit from a parent class. You get all the tools (fields) and instructions (methods) already provided, and you can add your own tools or modify the instructions to suit your needs.

### Purposes

- To reuse existing code without rewriting it
- To create hierarchical classifications of objects
- To enable polymorphism by allowing subclass objects to be treated as superclass objects
- To establish an "is-a" relationship between classes
- To facilitate extensibility by allowing new classes to build upon existing ones

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[accessModifier] class SubclassName extends SuperclassName {
    // additional fields and methods
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `accessModifier` | `public`, `protected`, `private`, or package-private. |
| `class` | Keyword introducing a class declaration. |
| `SubclassName` | The name of the new class. |
| `extends` | Keyword indicating inheritance. |
| `SuperclassName` | The name of the class being extended. |
| `class body` | Additional members and overridden methods. |

**Syntax Rules:**

- A class can extend only one superclass (single inheritance).
- If no `extends` clause is present, the class implicitly extends `Object`.
- A subclass inherits all non-private members of its superclass.
- Constructors are not inherited.

**Constraints and Limitations:**

- Java does not support multiple inheritance of classes (a class cannot extend more than one class).
- A `final` class cannot be extended.
- Private members of the superclass are not accessible in the subclass.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Inheritance**

```java
// File: InheritanceDemo.java
// Superclass
class Bicycle {
    public int cadence;
    public int gear;
    public int speed;

    public Bicycle(int startCadence, int startSpeed, int startGear) {
        gear = startGear;
        cadence = startCadence;
        speed = startSpeed;
    }

    public void setCadence(int newValue) {
        cadence = newValue;
    }

    public void speedUp(int increment) {
        speed += increment;
    }

    public void applyBrake(int decrement) {
        speed -= decrement;
    }
}

// Subclass
class MountainBike extends Bicycle {
    public int seatHeight;

    public MountainBike(int startHeight, int startCadence,
                        int startSpeed, int startGear) {
        super(startCadence, startSpeed, startGear);  // Call superclass constructor
        seatHeight = startHeight;
    }

    public void setHeight(int newValue) {
        seatHeight = newValue;
    }
}

public class InheritanceDemo {
    public static void main(String[] args) {
        MountainBike mb = new MountainBike(25, 10, 0, 1);
        mb.speedUp(20);
        mb.setHeight(30);
        System.out.println("Speed: " + mb.speed);         // 20
        System.out.println("Seat height: " + mb.seatHeight); // 30
    }
}
```

**Expected Output:**
```
Speed: 20
Seat height: 30
```

**Step-by-Step Explanation:**

1. `Bicycle` is the superclass with fields `cadence`, `gear`, `speed` and methods `setCadence`, `speedUp`, `applyBrake`.
2. `MountainBike` extends `Bicycle`, inheriting all its members.
3. `MountainBike` adds its own field `seatHeight` and method `setHeight`.
4. The `super(...)` call in `MountainBike`'s constructor invokes `Bicycle`'s constructor to initialize inherited fields.
5. `mb.speedUp(20)` uses the inherited method, and `mb.setHeight(30)` uses the subclass method.

### Real-World Cases

- **Java I/O:** `InputStream` is the superclass of `FileInputStream`, `ByteArrayInputStream`, etc.
- **Collections Framework:** `AbstractList` is extended by `ArrayList` and `LinkedList`.
- **GUI Frameworks:** `JComponent` is extended by `JButton`, `JLabel`, etc.

---

## 2. extends

### Definitions

**Core Definition:** The `extends` keyword is used in a class declaration to specify that the class is derived from (inherits from) another class.

**Technical Definition:** The `extends` clause in a class declaration specifies the direct superclass of the class being declared. The class inherits all members from its superclass, and the superclass's constructor is invoked during subclass object creation.

**Beginner-Friendly Explanation:** `extends` is like saying "this new class is a more specific version of that existing class." It tells Java: "Build this class on top of the existing one."

### Purposes

- To establish an inheritance relationship between two classes
- To specify which class's members are inherited
- To enable code reuse and hierarchical class design

### Syntax Rules and Structure

```java
class Subclass extends Superclass { ... }
```

**Syntax Rules:**

- The `extends` clause appears after the class name and before the class body.
- Only one superclass can be specified.
- The superclass must be accessible (not `private`).
- A class cannot extend itself or create circular inheritance.

**Constraints and Limitations:**

- A `final` class cannot appear in an `extends` clause.
- An interface uses `extends` to extend multiple interfaces.

### Annotated Code Example

```java
// File: ExtendsDemo.java
class Animal {
    public void eat() {
        System.out.println("Animal eats");
    }
}

class Dog extends Animal {
    public void bark() {
        System.out.println("Dog barks");
    }
}

public class ExtendsDemo {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.eat();   // Inherited from Animal
        dog.bark();  // Defined in Dog
    }
}
```

**Expected Output:**
```
Animal eats
Dog barks
```

**Step-by-Step Explanation:**

1. `Dog extends Animal` establishes the inheritance relationship.
2. `Dog` inherits `eat()` from `Animal`.
3. `Dog` adds its own method `bark()`.
4. The output shows that `dog` can call both inherited and own methods.

### Real-World Cases

- **`public class ArrayList<E> extends AbstractList<E>`**
- **`public class FileInputStream extends InputStream`**
- **`public class JButton extends AbstractButton`**

---

## 3. Superclass

### Definitions

**Core Definition:** A superclass is the class from which a subclass is derived; it is the parent or base class.

**Technical Definition:** The class from which the subclass is derived is called a superclass (also a base class or a parent class). Every class except `Object` has exactly one direct superclass.

**Beginner-Friendly Explanation:** A superclass is like a parent in a family. It provides the basic traits and behaviours that its children (subclasses) inherit.

### Purposes

- To provide common fields and methods that subclasses can reuse
- To define a general, abstract concept from which more specific classes derive
- To enable polymorphism by providing a common type

### Syntax Rules and Structure

```java
class Superclass {
    // common fields and methods
}

class Subclass extends Superclass {
    // additional members
}
```

**Syntax Rules:**

- A superclass can be abstract or concrete.
- The superclass's constructors are not inherited but can be called via `super()`.
- Private members are not accessible in the subclass.

**Constraints and Limitations:**

- A superclass cannot be `final` if it is intended to be extended.
- A superclass cannot be a primitive type or an interface (for classes).

### Annotated Code Example

```java
// File: SuperclassDemo.java
class Vehicle {
    protected String brand = "Generic";

    public void honk() {
        System.out.println("Beep!");
    }
}

class Car extends Vehicle {
    private String model = "Sedan";

    public void display() {
        System.out.println(brand + " " + model);  // Accessing inherited field
    }
}

public class SuperclassDemo {
    public static void main(String[] args) {
        Car car = new Car();
        car.honk();     // Beep!
        car.display();  // Generic Sedan
    }
}
```

**Expected Output:**
```
Beep!
Generic Sedan
```

**Step-by-Step Explanation:**

1. `Vehicle` is the superclass with a protected field `brand` and a method `honk()`.
2. `Car` extends `Vehicle` and adds a `model` field and `display()` method.
3. `Car` can access the inherited `brand` field because it is `protected`.

### Real-World Cases

- **`Object` is the superclass of all classes.**
- **`Number` is the superclass of `Integer`, `Double`, etc.**
- **`Throwable` is the superclass of `Exception` and `Error`.**

---

## 4. Subclass

### Definitions

**Core Definition:** A subclass is a class that is derived from another class, inheriting its members and extending or modifying its behaviour.

**Technical Definition:** A class that is derived from another class is called a subclass (also a derived class, extended class, or child class). A subclass inherits all the members (fields, methods, and nested classes) from its superclass. Constructors are not inherited.

**Beginner-Friendly Explanation:** A subclass is like a child in a family. It inherits traits from its parent (superclass) but can also have its own unique traits.

### Purposes

- To extend or specialize the behaviour of a superclass
- To add new fields and methods specific to the subclass
- To override inherited methods to provide different behaviour

### Syntax Rules and Structure

```java
class Subclass extends Superclass {
    // additional fields and methods
    // overridden methods
}
```

**Syntax Rules:**

- A subclass can access all non-private members of its superclass.
- A subclass can override inherited methods (except `final` methods).
- A subclass can call superclass constructors using `super(...)`.

**Constraints and Limitations:**

- A subclass cannot reduce the visibility of an inherited method.
- A subclass cannot override a `final` method.
- A subclass inherits `private` members but cannot access them directly.

### Annotated Code Example

```java
// File: SubclassDemo.java
class Shape {
    public double area() {
        return 0;
    }
}

class Circle extends Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class SubclassDemo {
    public static void main(String[] args) {
        Circle c = new Circle(5);
        System.out.println("Area: " + c.area());  // Area: 78.53981633974483
    }
}
```

**Expected Output:**
```
Area: 78.53981633974483
```

**Step-by-Step Explanation:**

1. `Shape` is the superclass with a generic `area()` method.
2. `Circle` extends `Shape` and overrides `area()` with a specific implementation.
3. `Circle` adds its own `radius` field.
4. The output shows the overridden method in action.

### Real-World Cases

- **`ArrayList` is a subclass of `AbstractList`.**
- **`FileInputStream` is a subclass of `InputStream`.**
- **`JButton` is a subclass of `AbstractButton`.**

---

## 5. Single Inheritance

### Definitions

**Core Definition:** Single inheritance is the restriction that a class can extend only one superclass.

**Technical Definition:** Excepting `Object`, which has no superclass, every class has one and only one direct superclass (single inheritance). Java does not support multiple inheritance of classes.

**Beginner-Friendly Explanation:** A child can have only one biological parent in Java's inheritance model. A class can inherit from only one other class.

### Purposes

- To avoid the complexity and ambiguity of multiple inheritance (the diamond problem)
- To provide a clear, predictable inheritance hierarchy
- To simplify the language and compiler design

### Syntax Rules and Structure

```java
class A { }
class B extends A { }  // OK: single inheritance
// class C extends A, B { }  // COMPILE ERROR: multiple inheritance not allowed
```

**Syntax Rules:**

- A class can extend exactly one superclass.
- A class can implement multiple interfaces (multiple inheritance of type).
- Interfaces can extend multiple interfaces.

**Constraints and Limitations:**

- Java does not support multiple inheritance of state or implementation.
- Some problems are harder to model with single inheritance (e.g., a class that needs behaviour from two sources).

### Annotated Code Example

```java
// File: SingleInheritanceDemo.java
class A {
    public void methodA() {
        System.out.println("Method A");
    }
}

class B extends A {
    public void methodB() {
        System.out.println("Method B");
    }
}

public class SingleInheritanceDemo {
    public static void main(String[] args) {
        B obj = new B();
        obj.methodA();  // Inherited from A
        obj.methodB();  // Defined in B
    }
}
```

**Expected Output:**
```
Method A
Method B
```

**Step-by-Step Explanation:**

1. `B` extends `A`, inheriting `methodA()`.
2. `B` adds `methodB()`.
3. The output shows both methods are accessible.

### Real-World Cases

- **Java Standard Library:** All classes form a single-inheritance hierarchy rooted at `Object`.
- **Frameworks:** Most frameworks use single inheritance to keep hierarchies manageable.

---

## 6. Method Overriding

### Definitions

**Core Definition:** Method overriding occurs when a subclass provides a new implementation for a method that is already defined in its superclass.

**Technical Definition:** An instance method in a subclass with the same signature (name, plus the number and the type of its parameters) and return type as an instance method in the superclass overrides the superclass's method. The ability of a subclass to override a method allows a class to inherit from a superclass whose behavior is "close enough" and then to modify behavior as needed. An overriding method can also return a subtype of the type returned by the overridden method; this subtype is called a covariant return type.

**Beginner-Friendly Explanation:** Overriding is like a child who inherits a recipe from a parent but decides to change some ingredients. The recipe name stays the same, but the child's version is used when you ask the child to cook.

### Purposes

- To allow a subclass to provide a specific implementation of a method already defined by its superclass
- To enable runtime polymorphism (dynamic method dispatch)
- To customize inherited behaviour for specific subclasses

### Syntax Rules and Structure

```java
// Superclass
public returnType methodName(parameters) { ... }

// Subclass
@Override
public returnType methodName(parameters) { ... }
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@Override` | Optional annotation that instructs the compiler to verify the method actually overrides a superclass method. |
| Same signature | Name and parameter types must match exactly. |
| Same or covariant return type | Return type must be the same or a subtype of the superclass method's return type. |
| Access modifier | Cannot be more restrictive than the overridden method's access modifier. |

**Syntax Rules:**

- The overriding method must have the same signature as the overridden method.
- The return type must be the same or a subtype (covariant return).
- The access modifier cannot be more restrictive.
- The overriding method cannot throw broader checked exceptions.
- `final`, `static`, and `private` methods cannot be overridden.

**Constraints and Limitations:**

- Constructors cannot be overridden.
- Static methods are hidden, not overridden.
- Private methods are not inherited and therefore cannot be overridden.

### Annotated Code Example

```java
// File: OverridingDemo.java
class Animal {
    public void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof! Woof!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

public class OverridingDemo {
    public static void main(String[] args) {
        Animal myDog = new Dog();  // Upcasting
        Animal myCat = new Cat();

        myDog.makeSound();  // Woof! Woof!
        myCat.makeSound();  // Meow!
    }
}
```

**Expected Output:**
```
Woof! Woof!
Meow!
```

**Step-by-Step Explanation:**

1. `Animal` defines `makeSound()`.
2. `Dog` and `Cat` override `makeSound()` with their own implementations.
3. `Animal myDog = new Dog()` upcasts the `Dog` object.
4. `myDog.makeSound()` invokes `Dog`'s version at runtime (dynamic dispatch).

### Real-World Cases

- **`toString()`:** Overridden in almost every custom class.
- **`equals()` and `hashCode()`:** Overridden for proper object equality.
- **`compareTo()`:** Overridden in classes implementing `Comparable`.

---

## 7. @Override

### Definitions

**Core Definition:** `@Override` is an annotation that instructs the compiler to verify that a method is intended to override a method in a superclass.

**Technical Definition:** When overriding a method, you might want to use the `@Override` annotation that instructs the compiler that you intend to override a method in the superclass. If, for some reason, the compiler detects that the method does not exist in one of the superclasses, then it will generate an error.

**Beginner-Friendly Explanation:** `@Override` is like a label that says "I am intentionally replacing my parent's method." If you make a mistake (e.g., typo in the method name), the compiler will warn you.

### Purposes

- To catch errors at compile time (e.g., misspelled method names)
- To document the programmer's intent
- To improve code readability

### Syntax Rules and Structure

```java
@Override
public returnType methodName(parameters) { ... }
```

**Syntax Rules:**

- The `@Override` annotation is placed immediately before the method declaration.
- It is optional but strongly recommended.
- It can be used on methods that override superclass methods or implement interface methods.

**Constraints and Limitations:**

- Using `@Override` on a method that does not override anything causes a compile-time error.
- `@Override` cannot be used on static methods (they are hidden, not overridden).

### Annotated Code Example

```java
// File: OverrideAnnotationDemo.java
class Parent {
    public void display() {
        System.out.println("Parent display");
    }
}

class Child extends Parent {
    @Override
    public void display() {  // Correct override
        System.out.println("Child display");
    }

    // @Override
    // public void display(int x) { }  // COMPILE ERROR: does not override
}

public class OverrideAnnotationDemo {
    public static void main(String[] args) {
        new Child().display();  // Child display
    }
}
```

**Expected Output:**
```
Child display
```

**Step-by-Step Explanation:**

1. `@Override` on `display()` verifies that it overrides `Parent.display()`.
2. If the method signature did not match, the compiler would produce an error.
3. This helps catch typos and signature mismatches early.

### Real-World Cases

- **IDE Support:** IDEs automatically insert `@Override` when generating overridden methods.
- **Static Analysis:** Tools like Checkstyle enforce the use of `@Override`.

---

## 8. super Calls

### Definitions

**Core Definition:** The `super` keyword is a reference to the superclass of the current object, used to access superclass members and constructors.

**Technical Definition:** If your method overrides one of its superclass's methods, you can invoke the overridden method through the use of the keyword `super`. `super` can also be used to refer to a hidden field. In a constructor, `super(...)` invokes a superclass constructor.

**Beginner-Friendly Explanation:** `super` is like saying "my parent" inside a subclass. It lets you call your parent's version of a method or constructor.

### Purposes

- To call a superclass constructor
- To access a superclass method that has been overridden
- To access a superclass field that has been hidden

### Syntax Rules and Structure

```java
super.fieldName;           // Access superclass field
super.methodName(args);    // Call superclass method
super(args);               // Call superclass constructor
```

**Syntax Rules:**

- `super(...)` must be the first statement in a constructor.
- `super.method()` can be used anywhere in an instance method.
- `super` cannot be used in static contexts.

**Constraints and Limitations:**

- `super` can only access the immediate superclass, not grandparent classes.
- A constructor cannot call both `this(...)` and `super(...)`.

### Annotated Code Example

```java
// File: SuperCallDemo.java
class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
        System.out.println("Animal constructor: " + name);
    }

    public void makeSound() {
        System.out.println("Some generic sound");
    }
}

class Dog extends Animal {
    public Dog(String name) {
        super(name);  // Calls Animal's constructor
        System.out.println("Dog constructor: " + name);
    }

    @Override
    public void makeSound() {
        super.makeSound();  // Calls Animal's version
        System.out.println("Woof!");
    }

    public static void main(String[] args) {
        Dog dog = new Dog("Rex");
        dog.makeSound();
    }
}
```

**Expected Output:**
```
Animal constructor: Rex
Dog constructor: Rex
Some generic sound
Woof!
```

**Step-by-Step Explanation:**

1. `super(name)` in `Dog` calls `Animal`'s constructor, which prints first.
2. Then `Dog`'s constructor body runs.
3. `super.makeSound()` calls `Animal`'s `makeSound` method, then `Dog`'s own code runs.

### Real-World Cases

- **Template Method Pattern:** `super.method()` calls a base implementation.
- **UI Components:** `super.paintComponent(g)` in Swing custom painting.

---

## 9. Constructor Chaining

### Definitions

**Core Definition:** Constructor chaining is the practice of one constructor calling another constructor, either in the same class (via `this()`) or in the superclass (via `super()`).

**Technical Definition:** Constructor chaining allows a constructor to reuse the initialization logic of another constructor. If a constructor does not explicitly call `this(...)` or `super(...)`, the compiler inserts a call to `super()`. The `super()` chain runs top-down — grandparent first, then parent, then child.

**Beginner-Friendly Explanation:** Constructor chaining is like a relay race. Each constructor passes the baton (initialization work) to the next one, ensuring that all the setup is done in the right order, from the top of the inheritance hierarchy down.

### Purposes

- To avoid duplicating initialization code between constructors
- To ensure superclass initialization happens before subclass initialization
- To provide a single "master" constructor that others delegate to

### Syntax Rules and Structure

```java
// In the same class:
public ClassName(args) {
    this(otherArgs);  // Must be first statement
    // additional code
}

// In a subclass:
public ClassName(args) {
    super(otherArgs);  // Must be first statement
    // additional code
}
```

**Syntax Rules:**

- `this(...)` and `super(...)` must be the first statement in a constructor.
- A constructor cannot call both `this(...)` and `super(...)`.
- At least one constructor in a class must call `super()` (explicitly or implicitly).

**Constraints and Limitations:**

- Circular constructor chaining causes a compile-time error.
- The `super()` call must match an accessible superclass constructor.

### Annotated Code Example

```java
// File: ConstructorChainingDemo.java
class Parent {
    protected String name;

    public Parent(String name) {
        this.name = name;
        System.out.println("Parent constructor: " + name);
    }
}

class Child extends Parent {
    private int age;

    public Child() {
        this("Unknown", 0);  // Calls Child's two-arg constructor
        System.out.println("Child no-arg constructor");
    }

    public Child(String name, int age) {
        super(name);  // Calls Parent's constructor
        this.age = age;
        System.out.println("Child two-arg constructor: " + name + ", " + age);
    }

    public static void main(String[] args) {
        new Child();
    }
}
```

**Expected Output:**
```
Parent constructor: Unknown
Child two-arg constructor: Unknown, 0
Child no-arg constructor
```

**Step-by-Step Explanation:**

1. `new Child()` invokes the no-arg constructor.
2. `this("Unknown", 0)` calls the two-arg constructor.
3. `super(name)` calls `Parent`'s constructor, which prints first.
4. Then the two-arg constructor body runs.
5. Finally, the no-arg constructor body runs.

### Real-World Cases

- **Inheritance Hierarchies:** Every subclass constructor calls `super()`.
- **Reducing Duplication:** A "master" constructor contains all initialization logic; others delegate to it.

---

## 10. Final Classes

### Definitions

**Core Definition:** A `final` class is a class that cannot be extended (subclassed).

**Technical Definition:** A class can be declared `final` if its definition is complete and no subclasses are desired or required. It is a compile-time error if the name of a `final` class appears in the `extends` clause of another class declaration; this implies that a `final` class cannot have any subclasses.

**Beginner-Friendly Explanation:** A `final` class is like a sealed box — you can use it, but you cannot modify it or build anything on top of it.

### Purposes

- To prevent unintended extension of a class
- To ensure the class's behaviour cannot be changed through subclassing
- To support immutability (e.g., `String` is `final`)
- To enable certain compiler optimizations

### Syntax Rules and Structure

```java
public final class ClassName {
    // class body
}
```

**Syntax Rules:**

- The `final` modifier is placed before the `class` keyword.
- A `final` class cannot be extended.
- A `final` class can still extend another class.

**Constraints and Limitations:**

- A class cannot be both `final` and `abstract` because `abstract` requires subclassing.
- All methods in a `final` class are implicitly `final` and cannot be overridden.

### Annotated Code Example

```java
// File: FinalClassDemo.java
final class ImmutablePoint {
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }
}

// class ExtendedPoint extends ImmutablePoint { }  // COMPILE ERROR

public class FinalClassDemo {
    public static void main(String[] args) {
        ImmutablePoint p = new ImmutablePoint(1, 2);
        System.out.println(p.getX() + ", " + p.getY());  // 1, 2
    }
}
```

**Expected Output:**
```
1, 2
```

**Step-by-Step Explanation:**

1. `ImmutablePoint` is declared `final`, so it cannot be extended.
2. The commented-out `ExtendedPoint` would cause a compile-time error.
3. The class can still be instantiated and used.

### Real-World Cases

- **`String`:** The `String` class is `final` to ensure immutability.
- **`Integer`, `Long`, `Double`:** Wrapper classes are `final`.
- **Security-Sensitive Classes:** Classes that perform security checks are often `final`.

---

## 11. Final Methods

### Definitions

**Core Definition:** A `final` method is a method that cannot be overridden by subclasses.

**Technical Definition:** A method can be declared `final` to prevent subclasses from overriding or hiding it. It is a compile-time error to attempt to override or hide a `final` method.

**Beginner-Friendly Explanation:** A `final` method is like a locked recipe — no one can change it, but everyone can use it.

### Purposes

- To prevent subclasses from changing critical behaviour
- To enable compiler optimizations
- To protect invariants

### Syntax Rules and Structure

```java
public final returnType methodName(parameters) { ... }
```

**Syntax Rules:**

- The `final` modifier is placed before the return type.
- A `final` method cannot be overridden.
- A `final` method can still be overloaded (a different signature).

**Constraints and Limitations:**

- The `final` modifier cannot be combined with `abstract`.
- `private` methods and methods in `final` classes are implicitly `final`.

### Annotated Code Example

```java
// File: FinalMethodDemo.java
class Base {
    public final void criticalOperation() {
        System.out.println("Critical operation — cannot be changed");
    }

    public void flexibleOperation() {
        System.out.println("Base flexible operation");
    }
}

class Derived extends Base {
    // @Override
    // public void criticalOperation() { }  // COMPILE ERROR

    @Override
    public void flexibleOperation() {
        System.out.println("Derived flexible operation");
    }
}

public class FinalMethodDemo {
    public static void main(String[] args) {
        Derived d = new Derived();
        d.criticalOperation();   // Critical operation — cannot be changed
        d.flexibleOperation();   // Derived flexible operation
    }
}
```

**Expected Output:**
```
Critical operation — cannot be changed
Derived flexible operation
```

**Step-by-Step Explanation:**

1. `criticalOperation()` is `final`, so `Derived` cannot override it.
2. `flexibleOperation()` is not `final`, so `Derived` successfully overrides it.

### Real-World Cases

- **`Object.getClass()`:** This method is `final` because the class of an object should never change.
- **Security-Sensitive Methods:** Methods performing security checks are often `final`.

---

## 12. Abstract Classes

### Definitions

**Core Definition:** An abstract class is a class that cannot be instantiated and is designed to be subclassed.

**Technical Definition:** An abstract class is a class that is declared `abstract` — it may or may not include abstract methods. Abstract classes cannot be instantiated, but they can be subclassed.

**Beginner-Friendly Explanation:** An abstract class is like a partially built house — you can't live in it yet (you can't instantiate it), but you can use it as a foundation to build complete houses (subclasses).

### Purposes

- To provide a common base class with shared code
- To define a contract that subclasses must fulfil
- To prevent direct instantiation of a general concept
- To enable code reuse across closely related classes

### Syntax Rules and Structure

```java
public abstract class ClassName {
    // fields
    // concrete methods
    // abstract methods
}
```

**Syntax Rules:**

- An abstract class is declared with the `abstract` keyword.
- It cannot be instantiated with `new`.
- It can have both abstract and concrete methods.
- If a class contains abstract methods, it must be declared abstract.
- Subclasses must implement all abstract methods or be declared abstract themselves.

**Constraints and Limitations:**

- An abstract class cannot be `final`.
- An abstract class can have constructors (called by subclasses).
- Abstract classes can have static fields and methods.

### Annotated Code Example

```java
// File: AbstractClassDemo.java
abstract class Shape {
    protected String name;

    public Shape(String name) {
        this.name = name;
    }

    // Abstract method: no body
    public abstract double area();

    // Concrete method
    public void display() {
        System.out.println(name + " area: " + area());
    }
}

class Circle extends Shape {
    private double radius;

    public Circle(double radius) {
        super("Circle");
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class AbstractClassDemo {
    public static void main(String[] args) {
        // Shape s = new Shape("Shape");  // COMPILE ERROR
        Circle c = new Circle(5);
        c.display();  // Circle area: 78.53981633974483
    }
}
```

**Expected Output:**
```
Circle area: 78.53981633974483
```

**Step-by-Step Explanation:**

1. `Shape` is abstract and cannot be instantiated.
2. `Circle` extends `Shape` and implements `area()`.
3. `c.display()` calls `Shape`'s concrete method, which calls the overridden `area()`.

### Real-World Cases

- **`AbstractList`, `AbstractMap`:** In the Collections Framework.
- **`InputStream`, `OutputStream`:** In Java I/O.
- **Template Method Pattern:** Abstract classes define the template method and abstract hooks.

---

## 13. abstract

### Definitions

**Core Definition:** The `abstract` keyword is a modifier that indicates a class or method is incomplete and must be extended or implemented.

**Technical Definition:** An abstract class is a class that is incomplete, or to be considered incomplete. It is a compile-time error if an attempt is made to create an instance of an abstract class using a class instance creation expression.

**Beginner-Friendly Explanation:** `abstract` is like saying "this is a concept, not a concrete thing." You can't have a "vehicle" (it's abstract), but you can have a "car" (concrete).

### Purposes

- To declare a class that cannot be instantiated
- To declare a method that must be implemented by subclasses
- To provide a base for a family of related classes

### Syntax Rules and Structure

```java
abstract class ClassName { ... }
abstract returnType methodName(parameters);
```

**Syntax Rules:**

- The `abstract` modifier can be applied to classes and methods.
- An abstract method has no body (ends with a semicolon).
- A class containing abstract methods must be declared abstract.

**Constraints and Limitations:**

- A class cannot be both `abstract` and `final`.
- An abstract method cannot be `private`, `static`, `final`, `native`, or `synchronized`.

### Annotated Code Example

```java
// File: AbstractKeywordDemo.java
abstract class Animal {
    public abstract void makeSound();

    public void sleep() {
        System.out.println("Sleeping...");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

public class AbstractKeywordDemo {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.makeSound();  // Woof!
        dog.sleep();      // Sleeping...
    }
}
```

**Expected Output:**
```
Woof!
Sleeping...
```

**Step-by-Step Explanation:**

1. `Animal` is abstract with an abstract `makeSound()` method and a concrete `sleep()` method.
2. `Dog` extends `Animal` and implements `makeSound()`.
3. `Dog` inherits `sleep()` and can use both methods.

### Real-World Cases

- **`HttpServlet`:** An abstract class with abstract methods like `doGet()`.
- **`AbstractCollection`:** Provides skeletal implementations of the `Collection` interface.

---

## 14. Abstract Methods

### Definitions

**Core Definition:** An abstract method is a method declared without an implementation, meant to be overridden by subclasses.

**Technical Definition:** An abstract method is a method that is declared without an implementation (without braces, and followed by a semicolon). If a class includes abstract methods, then the class itself must be declared abstract.

**Beginner-Friendly Explanation:** An abstract method is like a job description — it says what needs to be done, but not how. Each subclass must provide its own "how."

### Purposes

- To define a contract that subclasses must fulfil
- To enable polymorphism by guaranteeing that all concrete subclasses provide specific behaviour
- To support the Template Method design pattern

### Syntax Rules and Structure

```java
public abstract returnType methodName(parameters);
```

**Syntax Rules:**

- Abstract methods have no body; they end with a semicolon.
- They must be declared in an abstract class or an interface.
- Concrete subclasses must override all inherited abstract methods.

**Constraints and Limitations:**

- An abstract method cannot be `private`, `static`, `final`, `native`, or `synchronized`.
- A class containing one or more abstract methods must be declared `abstract`.

### Annotated Code Example

```java
// File: AbstractMethodDemo.java
abstract class Shape {
    public abstract double area();
}

class Rectangle extends Shape {
    private double length, width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    public double area() {
        return length * width;
    }
}

public class AbstractMethodDemo {
    public static void main(String[] args) {
        Rectangle r = new Rectangle(4, 3);
        System.out.println("Area: " + r.area());  // Area: 12.0
    }
}
```

**Expected Output:**
```
Area: 12.0
```

**Step-by-Step Explanation:**

1. `Shape` declares abstract `area()`.
2. `Rectangle` extends `Shape` and provides an implementation.
3. `r.area()` returns `4 * 3 = 12.0`.

### Real-World Cases

- **`AbstractList.get(int)`:** Abstract method that subclasses must implement.
- **`HttpServlet.doGet()`:** Abstract method for handling HTTP GET requests.
- **Template Method Pattern:** Abstract hooks are called by the template method.

---

## 15. Template Method Pattern

### Definitions

**Core Definition:** The Template Method pattern is a behavioural design pattern that defines the skeleton of an algorithm in a method, deferring some steps to subclasses.

**Technical Definition:** The Template Method design pattern is a solution to implement a general class of algorithms, leaving some placeholders for the parts of the algorithm that vary from one context to another. The pattern consists of implementing the common parts of the algorithm in a method (the template method) in an abstract class. The template method calls abstract "hook" methods that subclasses implement.

**Beginner-Friendly Explanation:** The Template Method pattern is like a recipe with some blanks to fill in. The recipe (template method) says: "First do this, then do the thing that varies, then do that." Each cook (subclass) fills in the blank with their own version.

### Purposes

- To define the skeleton of an algorithm in a base class
- To allow subclasses to redefine certain steps without changing the algorithm's structure
- To promote code reuse by putting common logic in the base class
- To enforce a consistent algorithm structure across subclasses

### Syntax Rules and Structure

```java
public abstract class AbstractClass {
    // Template method (often final)
    public final void templateMethod() {
        step1();
        step2();  // Abstract method
        step3();  // Abstract method
    }

    private void step1() { /* common implementation */ }

    protected abstract void step2();
    protected abstract void step3();
}
```

**Syntax Rules:**

- The template method is typically `final` to prevent overriding.
- Abstract "hook" methods are declared `abstract` and implemented by subclasses.
- Concrete methods in the base class provide common logic.

**Constraints and Limitations:**

- The pattern uses inheritance, which is a tight coupling.
- Too many hook methods can make the base class complex.

### Annotated Code Example

```java
// File: TemplateMethodDemo.java
abstract class DataProcessor {
    // Template method — defines the algorithm skeleton
    public final void process() {
        readData();
        transformData();
        writeData();
    }

    protected abstract void readData();
    protected abstract void transformData();
    protected abstract void writeData();
}

class CSVProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading CSV data");
    }

    @Override
    protected void transformData() {
        System.out.println("Transforming CSV data");
    }

    @Override
    protected void writeData() {
        System.out.println("Writing CSV data");
    }
}

public class TemplateMethodDemo {
    public static void main(String[] args) {
        CSVProcessor processor = new CSVProcessor();
        processor.process();
    }
}
```

**Expected Output:**
```
Reading CSV data
Transforming CSV data
Writing CSV data
```

**Step-by-Step Explanation:**

1. `DataProcessor` defines the template method `process()` with the algorithm skeleton.
2. Abstract methods `readData()`, `transformData()`, and `writeData()` are hooks.
3. `CSVProcessor` implements the hooks.
4. `processor.process()` calls the template method, which invokes the overridden hooks.

### Real-World Cases

- **`HttpServlet.service()`:** Template method that calls `doGet()`, `doPost()`, etc.
- **`InputStream.read()`:** Template method with abstract `read(byte[], int, int)`.
- **`AbstractList.indexOf()`:** Template method that uses abstract `get(int)`.

---

## 16. Inheritance Best Practices

### Definitions

**Core Definition:** Inheritance best practices are guidelines for using inheritance effectively and avoiding its pitfalls.

**Technical Definition:** Best practices derive from *Effective Java* and the SEI CERT Oracle Coding Standard. Key practices include designing for inheritance or prohibiting it, favouring composition over inheritance, and documenting the inheritance contract.

**Beginner-Friendly Explanation:** Inheritance best practices are like the rules for building a safe house — use the right materials, follow the building code, and don't cut corners.

### Purposes

- To avoid fragile base class problems
- To ensure subclasses can be extended safely
- To reduce coupling between classes
- To maintain encapsulation

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Design for inheritance or prohibit it | Document and test subclass behaviour, or make the class `final`. |
| Favour composition over inheritance | Use "has-a" instead of "is-a" when possible. |
| Provide protected hooks | Allow subclasses to customize behaviour safely. |
| Avoid exposing internal details | Do not make protected fields or methods that expose implementation. |
| Document the contract | Specify how methods behave and what subclasses must not do. |
| Use `@Override` | Catch errors at compile time. |
| Keep hierarchies shallow | Deep hierarchies are hard to understand and maintain. |

**Syntax Rules:**

- Use `final` on classes that should not be extended.
- Use `@Override` on all overriding methods.
- Document self-use of overridable methods.

**Constraints and Limitations:**

- Inheritance is a tight coupling mechanism.
- Overuse of inheritance can lead to rigid designs.

### Annotated Code Example

```java
// File: BestPracticesInheritance.java
// Option 1: Design for inheritance with documentation
public abstract class AbstractShape {
    /**
     * Returns the area of this shape.
     * @return the area in square units
     */
    public abstract double area();

    /**
     * Hook for subclasses to perform additional validation.
     * Default implementation does nothing.
     */
    protected void validate() { }
}

// Option 2: Prohibit inheritance with final
public final class ImmutablePoint {
    private final int x, y;
    public ImmutablePoint(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
    public int getY() { return y; }
}

public class BestPracticesInheritance {
    public static void main(String[] args) {
        System.out.println("Best practices demonstrated.");
    }
}
```

**Expected Output:**
```
Best practices demonstrated.
```

**Step-by-Step Explanation:**

1. `AbstractShape` is designed for inheritance with documented abstract methods and a protected hook.
2. `ImmutablePoint` is `final`, prohibiting inheritance because its definition is complete.
3. This demonstrates the two options: design for inheritance or prohibit it.

### Real-World Cases

- **`java.util.AbstractList`:** Designed for inheritance with protected hooks.
- **`String`:** `final` to prevent subclassing and ensure immutability.
- **`Effective Java`:** Recommends composition over inheritance for flexibility.

---

## 17. Composition over Inheritance

### Definitions

**Core Definition:** Composition over inheritance is a design principle that favours using "has-a" relationships (composition) over "is-a" relationships (inheritance) for code reuse.

**Technical Definition:** Composition involves creating a new class that contains a private field referencing an instance of an existing class. This design is called composition because the existing class becomes a component of the new one. The SEI CERT guideline recommends using composition rather than inheritance to achieve the desired effect.

**Beginner-Friendly Explanation:** Composition is like building a car by assembling parts (wheels, engine, seats) rather than inheriting from a "vehicle" class. Instead of saying "a car is a vehicle," you say "a car has an engine."

### Purposes

- To reduce tight coupling between classes
- To allow behaviour reuse without the limitations of single inheritance
- To maintain encapsulation (the composed object's internals are not exposed)
- To make designs more flexible and easier to change

### Syntax Rules and Structure

```java
public class ComposedClass {
    private ExistingClass component;  // Composition: "has-a"

    public ComposedClass(ExistingClass component) {
        this.component = component;
    }

    // Delegate methods to the component
    public returnType method() {
        return component.method();
    }
}
```

**Syntax Rules:**

- The composed class holds a private reference to the existing class.
- Methods delegate to the component's methods.
- The composed class can add its own behaviour.

**Constraints and Limitations:**

- Composition requires more code (delegation methods).
- The composed class must be designed to be used as a component.

### Annotated Code Example

```java
// File: CompositionOverInheritance.java
// Existing class
class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

// Composition: Car HAS-A Engine
class Car {
    private Engine engine = new Engine();  // Composition

    public void startCar() {
        engine.start();  // Delegation
        System.out.println("Car is ready to drive");
    }
}

// Inheritance for comparison
class Vehicle {
    public void move() {
        System.out.println("Vehicle moving");
    }
}

class SportsCar extends Vehicle {
    @Override
    public void move() {
        System.out.println("Sports car moving fast");
    }
}

public class CompositionOverInheritance {
    public static void main(String[] args) {
        Car car = new Car();
        car.startCar();
        // Output:
        // Engine started
        // Car is ready to drive

        SportsCar sc = new SportsCar();
        sc.move();
        // Output: Sports car moving fast
    }
}
```

**Expected Output:**
```
Engine started
Car is ready to drive
Sports car moving fast
```

**Step-by-Step Explanation:**

1. `Car` uses composition: it has an `Engine` field.
2. `Car.startCar()` delegates to `engine.start()` and adds its own behaviour.
3. `SportsCar` uses inheritance: it extends `Vehicle` and overrides `move()`.
4. Composition provides flexibility (you can swap engines); inheritance creates a fixed relationship.

### Real-World Cases

- **`java.util.Properties`:** Composition is favoured over inheriting from `Hashtable`.
- **Strategy Pattern:** Uses composition to swap algorithms at runtime.
- **Decorator Pattern:** Uses composition to add behaviour dynamically.

---

## References Links

- Inheritance — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html
- Overriding and Hiding Methods — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/override.html
- Abstract Methods and Classes — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html
- Using the Keyword super — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/super.html
- Summary of Inheritance — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/summaryinherit.html
- Java Language Specification, §8.1.1.2: final Classes - https://docs.oracle.com/javase/specs/jls/se16/preview/specs/local-statics-jls.html
- Java Language Specification, §8.4.3.1: abstract Methods - https://docs.oracle.com/javase/specs/jls/se16/html/jls-8.html
- Java Language Specification, §8.4.8.1: Overriding (by Instance Methods) - https://docs.oracle.com/javase/specs/jls/se16/html/jls-8.html
- SEI CERT Oracle Coding Standard — MET52-J: Do not use the clone method to copy untrusted method parameters - https://wiki.sei.cmu.edu/confluence/display/java/MET52-JG.+Do+not+use+the+clone+method+to+copy+untrusted+method+parameters
- SEI CERT Oracle Coding Standard — OBJ09-J: Defensively copy private mutable class members before returning their references - https://wiki.sei.cmu.edu/confluence/display/java/OBJ09-J.+Defensively+copy+private+mutable+class+members+before+returning+their+references
- Effective Java (3rd Edition) by Joshua Bloch — Item 18: Favor composition over inheritance
- Effective Java (3rd Edition) by Joshua Bloch — Item 19: Design and document for inheritance or else prohibit it
- Template Method Pattern — Refactoring Guru - https://refactoring.guru/design-patterns/template-method
- Composition over Inheritance — Java Magazine (Oracle) - https://blogs.oracle.com/javamagazine/post/java-inheritance-composition