# Java Abstract Classes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An abstract class is a class that is declared with the `abstract` keyword and cannot be instantiated. It serves as a partial blueprint for subclasses, providing common state and behaviour while leaving certain methods unimplemented for subclasses to complete.

**Technical Definition:** Per the Java Language Specification, "A named class may be declared `abstract` and must be declared `abstract` if it is incompletely implemented; such a class cannot be instantiated, but can be extended by subclasses". An abstract class may or may not include abstract methods. "An abstract method is a method that is declared without an implementation (without braces, and followed by a semicolon)". Every subclass of an abstract class that is not itself abstract must provide implementations for all inherited abstract methods, or a compile-time error occurs.

**Beginner-Friendly Explanation:** An abstract class is like a partially built house. You cannot live in it yet (you cannot instantiate it), but it provides the foundation, walls, and plumbing (common fields and concrete methods). Builders (subclasses) can complete the house by adding the remaining parts (implementing abstract methods). Each builder might finish the house differently, but they all start from the same basic structure.

### Key Characteristics

- **Cannot Be Instantiated:** Abstract classes cannot be instantiated with `new`, but they can be subclassed.
- **Partial Implementation:** They may contain a mix of abstract methods (no body) and concrete methods (with body).
- **Stateful:** Unlike interfaces, abstract classes can declare non-static, non-final fields, enabling them to maintain state.
- **Constructors:** Abstract classes can have constructors, which are invoked when a concrete subclass is instantiated.
- **Single Inheritance:** A class can extend only one abstract class, whereas it can implement multiple interfaces.
- **Access Modifiers:** Concrete methods in abstract classes can use any access modifier (`public`, `protected`, `private`), unlike interface methods which are implicitly public.

### Prerequisites

- Basic understanding of Java classes and objects
- Familiarity with inheritance and the `extends` keyword
- Knowledge of method overriding and the `@Override` annotation
- Awareness of access modifiers and their scope

### Related Programming Areas

- **Object-Oriented Programming (OOP):** Abstract classes are a key mechanism for abstraction and code reuse.
- **Design Patterns:** The Template Method pattern relies on abstract classes to define algorithm skeletons.
- **Frameworks:** The Java Collections Framework uses abstract classes like `AbstractList` and `AbstractMap` to provide skeletal implementations.
- **GUI Programming:** Swing's `JComponent` is an abstract class providing common behaviour for all UI components.
- **I/O:** `InputStream` and `OutputStream` are abstract classes at the root of Java's I/O hierarchy.

### Core Concepts / Features

---

## 1. Abstract Classes

### Definitions

**Core Definition:** An abstract class is a class declared with the `abstract` keyword that cannot be instantiated and is designed to be extended by subclasses.

**Technical Definition:** "An abstract class is a class that is declared `abstract`—it may or may not include abstract methods. Abstract classes cannot be instantiated, but they can be subclassed". The JLS states: "A named class may be declared `abstract` and must be declared `abstract` if it is incompletely implemented". An abstract class provides a common base for closely related classes, allowing shared code to be defined once while deferring specific behaviour to subclasses.

**Beginner-Friendly Explanation:** An abstract class is like a template for building related objects. It says: "All objects of this type should have these things in common, but some details will be filled in by each specific type." You cannot create an object from an abstract class directly — you must create a concrete subclass first.

### Purposes

- To provide a common base class with shared code for closely related classes
- To define a contract (abstract methods) that subclasses must fulfil
- To prevent direct instantiation of a general concept that is too vague to exist on its own
- To enable code reuse across a family of related classes
- To support the Template Method pattern by defining algorithm skeletons

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[accessModifier] abstract class ClassName [extends Superclass] [implements Interfaces] {
    // fields (instance or static)
    // constructors
    // concrete methods
    // abstract methods
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abstract` | Keyword indicating the class is abstract. |
| `ClassName` | A legal Java identifier, capitalized by convention. |
| `extends` | Optional; specifies the superclass. |
| `implements` | Optional; specifies interfaces. |
| Fields | Can be instance or static, with any access modifier. |
| Constructors | Used to initialize inherited state. |
| Concrete methods | Methods with implementations. |
| Abstract methods | Method declarations without bodies. |

**Syntax Rules:**

- The `abstract` modifier is placed before the `class` keyword.
- An abstract class cannot be instantiated with `new`.
- If a class contains one or more abstract methods, it must be declared abstract.
- A subclass must implement all inherited abstract methods or be declared abstract itself.
- An abstract class can override an abstract method by providing another abstract method declaration.

**Constraints and Limitations:**

- An abstract class cannot be declared `final` (final classes cannot be subclassed).
- An abstract method cannot be `private`, `static`, `final`, `native`, or `synchronized`.
- Abstract classes cannot be instantiated even if they contain no abstract methods.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Abstract Class with Abstract and Concrete Methods**

```java
// File: AbstractClassDemo.java
// Step 1: Declare an abstract class
abstract class GraphicObject {
    // Instance fields (state)
    protected int x, y;
    protected String color;

    // Constructor: initializes common state
    public GraphicObject(int x, int y, String color) {
        this.x = x;
        this.y = y;
        this.color = color;
    }

    // Concrete method: shared behaviour
    public void moveTo(int newX, int newY) {
        this.x = newX;
        this.y = newY;
        System.out.println("Moved to (" + x + ", " + y + ")");
    }

    // Abstract method: must be implemented by subclasses
    public abstract void draw();

    // Abstract method: must be implemented by subclasses
    public abstract double area();
}

// Step 2: Concrete subclass implements abstract methods
class Circle extends GraphicObject {
    private double radius;

    public Circle(int x, int y, String color, double radius) {
        super(x, y, color);  // Call abstract class constructor
        this.radius = radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " circle at (" + x + ", " + y + ")");
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

// Step 3: Another concrete subclass
class Rectangle extends GraphicObject {
    private double width, height;

    public Rectangle(int x, int y, String color, double width, double height) {
        super(x, y, color);
        this.width = width;
        this.height = height;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " rectangle at (" + x + ", " + y + ")");
    }

    @Override
    public double area() {
        return width * height;
    }
}

public class AbstractClassDemo {
    public static void main(String[] args) {
        // GraphicObject obj = new GraphicObject(0, 0, "red");  // COMPILE ERROR

        // Polymorphic references
        GraphicObject circle = new Circle(10, 20, "red", 5);
        GraphicObject rect = new Rectangle(30, 40, "blue", 4, 3);

        // Polymorphic method calls
        circle.draw();      // Drawing a red circle at (10, 20)
        circle.moveTo(50, 60);  // Moved to (50, 60)
        System.out.println("Circle area: " + circle.area());  // 78.53981633974483

        rect.draw();        // Drawing a blue rectangle at (30, 40)
        System.out.println("Rectangle area: " + rect.area());  // 12.0
    }
}
```

**Expected Output:**
```
Drawing a red circle at (10, 20)
Moved to (50, 60)
Circle area: 78.53981633974483
Drawing a blue rectangle at (30, 40)
Rectangle area: 12.0
```

**Step-by-Step Explanation:**

1. `GraphicObject` is declared `abstract`, so it cannot be instantiated.
2. It contains instance fields (`x`, `y`, `color`), a constructor, a concrete method (`moveTo`), and two abstract methods (`draw`, `area`).
3. `Circle` and `Rectangle` extend `GraphicObject` and implement the abstract methods.
4. The `super(...)` call in each subclass constructor invokes `GraphicObject`'s constructor to initialize inherited state.
5. In `main`, `GraphicObject` references point to `Circle` and `Rectangle` objects (polymorphism).
6. `circle.draw()` and `rect.draw()` invoke the overridden implementations at runtime.
7. `circle.moveTo(50, 60)` calls the inherited concrete method.
8. `circle.area()` and `rect.area()` invoke the overridden abstract method implementations.

### Real-World Cases

- **`AbstractMap`:** Provides skeletal implementation of the `Map` interface; `HashMap` and `TreeMap` extend it.
- **`AbstractList`:** Provides skeletal implementation of the `List` interface; `ArrayList` and `LinkedList` extend it.
- **`InputStream`:** Abstract class at the root of the byte input stream hierarchy.
- **`HttpServlet`:** Abstract class that servlets extend to handle HTTP requests.
- **`JComponent`:** Abstract class that provides common behaviour for Swing components.

---

## 2. abstract Keyword

### Definitions

**Core Definition:** The `abstract` keyword is a non-access modifier used to declare classes and methods that are incomplete and must be extended or implemented.

**Technical Definition:** "The `abstract` keyword is a non-access modifier, used for classes and methods. Class: An abstract class is a restricted class that cannot be used to create objects (to access it, it must be inherited from another class). Method: An abstract method can only be used in an abstract class, and it does not have a body". The JLS states that "a method whose declaration does not indicate how it is implemented must be declared `abstract`".

**Beginner-Friendly Explanation:** `abstract` is like the word "TBD" (to be determined) on a form. It means "this part is not filled in yet — someone else will fill it in later." For classes, it means you can't use it directly. For methods, it means the method has no instructions yet.

### Purposes

- To declare a class that cannot be instantiated
- To declare a method that must be implemented by subclasses
- To indicate that a class is incomplete and requires subclassing
- To establish a contract for subclasses to fulfil

### Syntax Rules and Structure

```java
// Abstract class
[accessModifier] abstract class ClassName { ... }

// Abstract method (no body)
[accessModifier] abstract returnType methodName(parameters);
```

**Syntax Rules:**

- The `abstract` modifier is placed before the `class` keyword for classes.
- For methods, `abstract` is placed before the return type.
- An abstract method has no body; it ends with a semicolon.
- A class containing abstract methods must be declared abstract.
- Constructors cannot be marked `abstract`.
- Static methods cannot be `abstract`.

**Constraints and Limitations:**

- An abstract class cannot be `final`.
- An abstract method cannot be `private`, `static`, `final`, `native`, or `synchronized`.
- The `abstract` modifier for interface methods is obsolete and should not be used.

### Annotated Code Example

```java
// File: AbstractKeywordDemo.java
// Abstract class
abstract class Vehicle {
    protected String brand;

    public Vehicle(String brand) {
        this.brand = brand;
    }

    // Abstract method: subclasses MUST implement
    public abstract void startEngine();

    // Concrete method: subclasses inherit
    public void displayBrand() {
        System.out.println("Brand: " + brand);
    }
}

class Car extends Vehicle {
    public Car(String brand) {
        super(brand);
    }

    @Override
    public void startEngine() {
        System.out.println("Car engine started");
    }
}

public class AbstractKeywordDemo {
    public static void main(String[] args) {
        // Vehicle v = new Vehicle("Generic");  // COMPILE ERROR

        Car car = new Car("Toyota");
        car.displayBrand();   // Brand: Toyota
        car.startEngine();    // Car engine started
    }
}
```

**Expected Output:**
```
Brand: Toyota
Car engine started
```

**Step-by-Step Explanation:**

1. `Vehicle` is `abstract` and contains an abstract method `startEngine()`.
2. `Car` extends `Vehicle` and implements `startEngine()`.
3. `Vehicle` cannot be instantiated, but `Car` can.
4. The output shows the inherited concrete method and the implemented abstract method.

### Real-World Cases

- **`abstract class AbstractList<E>`:** Provides skeletal implementation of `List`.
- **`abstract void draw()`:** In graphics frameworks, abstract methods force subclasses to define rendering.
- **`abstract class HttpServlet`:** Forces servlets to implement `doGet` or `doPost`.

---

## 3. Abstract Methods

### Definitions

**Core Definition:** An abstract method is a method declared without an implementation, meant to be overridden by subclasses.

**Technical Definition:** "An abstract method declaration introduces the method as a member, providing its signature, result, and throws clause if any, but does not provide an implementation". "A method that is not abstract may be referred to as a concrete method". Every concrete subclass of an abstract class containing abstract methods must provide an implementation for each abstract method, or a compile-time error occurs.

**Beginner-Friendly Explanation:** An abstract method is like a job description — it says what needs to be done, but not how. Each subclass must provide its own "how."

### Purposes

- To define a contract that subclasses must fulfil
- To enable polymorphism by guaranteeing that all concrete subclasses provide specific behaviour
- To support the Template Method design pattern
- To defer implementation details to subclasses

### Syntax Rules and Structure

```java
[accessModifier] abstract returnType methodName(parameters) [throws exceptions];
```

**Syntax Rules:**

- No body; ends with a semicolon.
- Must be declared in an abstract class or interface.
- Concrete subclasses must override all inherited abstract methods.
- An abstract class can override an abstract method by providing another abstract method declaration.
- An instance method that is not abstract can be overridden by an abstract method.

**Constraints and Limitations:**

- Cannot be `private`, `static`, `final`, `native`, or `synchronized`.
- Cannot have a body (no braces).
- A class containing abstract methods must be abstract.

### Annotated Code Example

```java
// File: AbstractMethodDemo.java
abstract class Shape {
    // Abstract methods: no body
    public abstract double area();
    public abstract double perimeter();

    // Concrete method
    public void describe() {
        System.out.println("Area: " + area() + ", Perimeter: " + perimeter());
    }
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

    @Override
    public double perimeter() {
        return 2 * (length + width);
    }
}

public class AbstractMethodDemo {
    public static void main(String[] args) {
        Rectangle rect = new Rectangle(4, 3);
        rect.describe();  // Area: 12.0, Perimeter: 14.0
    }
}
```

**Expected Output:**
```
Area: 12.0, Perimeter: 14.0
```

**Step-by-Step Explanation:**

1. `Shape` declares two abstract methods: `area()` and `perimeter()`.
2. `Rectangle` extends `Shape` and implements both methods.
3. `rect.describe()` calls the concrete method, which invokes the implemented abstract methods.

### Real-World Cases

- **`AbstractList.get(int index)`:** Abstract method that subclasses must implement.
- **`HttpServlet.doGet()`:** Abstract method for handling HTTP GET requests.
- **`InputStream.read()`:** Abstract method that subclasses implement for specific input sources.

---

## 4. Concrete Methods

### Definitions

**Core Definition:** A concrete method is a method within an abstract class (or any class) that has a complete implementation.

**Technical Definition:** "A method that is not abstract may be referred to as a concrete method". Concrete methods in abstract classes provide shared behaviour that subclasses inherit and can use directly, or override if needed.

**Beginner-Friendly Explanation:** A concrete method is a method that actually does something. It has instructions (a body) that subclasses can use as-is or replace.

### Purposes

- To provide shared behaviour that all subclasses inherit
- To reduce code duplication by centralizing common logic
- To offer default implementations that subclasses can use or override
- To define utility methods within the abstract class

### Syntax Rules and Structure

```java
[accessModifier] returnType methodName(parameters) {
    // implementation
}
```

**Syntax Rules:**

- Has a body enclosed in braces.
- Can use any access modifier (`public`, `protected`, `private`).
- Can be overridden by subclasses unless declared `final`.
- Can call abstract methods (which will be dispatched to subclass implementations at runtime).

**Constraints and Limitations:**

- A concrete method cannot be in an interface (except default methods).
- If declared `final`, it cannot be overridden.

### Annotated Code Example

```java
// File: ConcreteMethodDemo.java
abstract class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    // Abstract method
    public abstract void makeSound();

    // Concrete method: shared behaviour
    public void eat() {
        System.out.println(name + " is eating");
    }

    // Concrete method that calls an abstract method
    public void introduce() {
        System.out.print(name + " says: ");
        makeSound();  // Dispatched to subclass implementation
    }
}

class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

public class ConcreteMethodDemo {
    public static void main(String[] args) {
        Dog dog = new Dog("Rex");
        dog.eat();        // Rex is eating
        dog.introduce();  // Rex says: Woof!
    }
}
```

**Expected Output:**
```
Rex is eating
Rex says: Woof!
```

**Step-by-Step Explanation:**

1. `Animal` defines a concrete `eat()` method and a concrete `introduce()` method.
2. `introduce()` calls the abstract `makeSound()`, which is implemented by `Dog`.
3. `dog.eat()` uses the inherited concrete method.
4. `dog.introduce()` calls the concrete method, which dispatches to `Dog.makeSound()`.

### Real-World Cases

- **`AbstractMap.get()`:** Concrete method that relies on abstract `entrySet()`.
- **`AbstractList.indexOf()`:** Concrete method that uses abstract `get(int)`.
- **`HttpServlet.service()`:** Concrete method that dispatches to `doGet()`, `doPost()`, etc.

---

## 5. Constructors

### Definitions

**Core Definition:** A constructor in an abstract class is a special method that initializes the abstract class's fields when a concrete subclass is instantiated.

**Technical Definition:** "Even though we cannot instantiate an abstract class directly using the `new` keyword, it can contain constructors. These constructors are called automatically when a concrete child class is instantiated, ensuring the parent class's state is properly initialized". "A constructor in Java doesn't actually 'build' the object, it is used to initialize fields".

**Beginner-Friendly Explanation:** An abstract class constructor is like the foundation of a building. You can't live in the foundation alone, but every building (subclass) starts with it. When a subclass is created, the abstract class's constructor runs first to set up the common parts.

### Purposes

- To initialize fields declared in the abstract class
- To enforce that subclasses provide required initialization data
- To avoid duplicating initialization code across subclasses
- To enforce invariants at the base class level

### Syntax Rules and Structure

```java
[accessModifier] AbstractClassName(parameters) {
    // initialization code
}
```

**Syntax Rules:**

- Constructors in abstract classes are declared like those in concrete classes.
- They are invoked via `super(...)` from subclass constructors.
- They can be `public`, `protected`, or package-private (but typically `protected`).
- They cannot be `private` if subclasses need to call them (though `private` is allowed if no subclass needs it).

**Constraints and Limitations:**

- Constructors cannot be `abstract`.
- An abstract class constructor is never called directly with `new`; it is always invoked through a subclass constructor.

### Annotated Code Example

```java
// File: AbstractConstructorDemo.java
abstract class Employee {
    protected String name;
    protected int id;

    // Abstract class constructor
    public Employee(String name, int id) {
        this.name = name;
        this.id = id;
        System.out.println("Employee constructor called: " + name + ", " + id);
    }

    public abstract double calculateSalary();
}

class Developer extends Employee {
    private double hourlyRate;
    private int hoursWorked;

    public Developer(String name, int id, double hourlyRate, int hoursWorked) {
        super(name, id);  // Calls abstract class constructor
        this.hourlyRate = hourlyRate;
        this.hoursWorked = hoursWorked;
        System.out.println("Developer constructor called");
    }

    @Override
    public double calculateSalary() {
        return hourlyRate * hoursWorked;
    }
}

public class AbstractConstructorDemo {
    public static void main(String[] args) {
        Developer dev = new Developer("Alice", 101, 50.0, 160);
        System.out.println("Salary: $" + dev.calculateSalary());
    }
}
```

**Expected Output:**
```
Employee constructor called: Alice, 101
Developer constructor called
Salary: $8000.0
```

**Step-by-Step Explanation:**

1. `Employee` has a constructor that initializes `name` and `id`.
2. `Developer` extends `Employee` and calls `super(name, id)` in its constructor.
3. When `new Developer(...)` is executed, the `Employee` constructor runs first.
4. Then the `Developer` constructor body runs.
5. `calculateSalary()` uses the inherited fields and the subclass-specific fields.

### Real-World Cases

- **`AbstractList(int initialCapacity)`:** Protected constructor used by `ArrayList`.
- **`HttpServlet()`:** No-arg constructor called by the servlet container.
- **`InputStream()`:** No-arg constructor called by all input stream subclasses.

---

## 6. Fields

### Definitions

**Core Definition:** Fields in an abstract class are variables that store state, which can be instance variables (per-object) or static variables (per-class).

**Technical Definition:** "With abstract classes, you can declare fields that are not static and final". Unlike interfaces (where fields are implicitly `public static final`), abstract classes can have instance fields with any access modifier, enabling them to maintain state that subclasses inherit and can modify.

**Beginner-Friendly Explanation:** Fields in an abstract class are like the common furniture in a shared house. Every room (subclass) gets the same basic furniture (fields), but each room can arrange or decorate it differently.

### Purposes

- To store state shared by all subclasses
- To provide common data that subclasses can access and modify
- To enable methods (concrete or abstract) to operate on shared state
- To support encapsulation with appropriate access modifiers

### Syntax Rules and Structure

```java
[accessModifier] [static] [final] type fieldName [= initialValue];
```

**Syntax Rules:**

- Abstract classes can have instance fields (non-static, non-final).
- Fields can have any access modifier: `public`, `protected`, `private`, or package-private.
- Fields can be `static`, `final`, or both.
- Fields can be initialized at declaration or in constructors.

**Constraints and Limitations:**

- Instance fields cannot be `final` unless initialized in every constructor.
- Static fields are shared across all subclasses.

### Annotated Code Example

```java
// File: AbstractFieldsDemo.java
abstract class BankAccount {
    // Instance fields (state)
    protected String accountNumber;
    protected double balance;

    // Static field (shared constant)
    public static final double MIN_BALANCE = 100.0;

    // Static field (shared counter)
    private static int accountCount = 0;

    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
        accountCount++;
        System.out.println("Account created: " + accountNumber + " (Total: " + accountCount + ")");
    }

    public abstract void deposit(double amount);

    public void displayBalance() {
        System.out.println(accountNumber + " balance: $" + balance);
    }
}

class SavingsAccount extends BankAccount {
    private double interestRate;

    public SavingsAccount(String accountNumber, double initialBalance, double interestRate) {
        super(accountNumber, initialBalance);
        this.interestRate = interestRate;
    }

    @Override
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited $" + amount + " to savings");
        }
    }
}

public class AbstractFieldsDemo {
    public static void main(String[] args) {
        SavingsAccount acc1 = new SavingsAccount("SA-001", 500.0, 0.02);
        SavingsAccount acc2 = new SavingsAccount("SA-002", 1000.0, 0.03);

        acc1.deposit(200.0);  // Deposited $200.0 to savings
        acc1.displayBalance();  // SA-001 balance: $700.0

        System.out.println("Min balance: " + BankAccount.MIN_BALANCE);  // 100.0
    }
}
```

**Expected Output:**
```
Account created: SA-001 (Total: 1)
Account created: SA-002 (Total: 2)
Deposited $200.0 to savings
SA-001 balance: $700.0
Min balance: 100.0
```

**Step-by-Step Explanation:**

1. `BankAccount` declares instance fields `accountNumber` and `balance`, plus static fields `MIN_BALANCE` and `accountCount`.
2. `SavingsAccount` inherits the instance fields and can access them directly (because they are `protected`).
3. The static counter increments each time an account is created.
4. `acc1.deposit(200.0)` modifies the inherited `balance` field.
5. `acc1.displayBalance()` uses the inherited concrete method to display the updated balance.

### Real-World Cases

- **`AbstractList`:** Has a `modCount` field for tracking structural modifications.
- **`AbstractMap`:** Has a `keySet` and `values` field for caching views.
- **`HttpServlet`:** Has no instance fields (stateless by design).

---

## 7. Abstract Class vs Interface

### Definitions

**Core Definition:** An abstract class is a class that can have state and partial implementation, while an interface defines a pure contract with no instance state.

**Technical Definition:** "Abstract classes are similar to interfaces. You cannot instantiate them, and they may contain a mix of methods declared with or without an implementation. However, with abstract classes, you can declare fields that are not static and final, and define public, protected, and private concrete methods. With interfaces, all fields are automatically public, static, and final, and all methods that you declare or define (as default methods) are public. In addition, you can extend only one class, whether or not it is abstract, whereas you can implement any number of interfaces".

**Beginner-Friendly Explanation:** An abstract class is like a partially built house — you can inherit the foundation and walls. An interface is like a blueprint — it only tells you what the house should have, not how to build it.

### Purposes

- To choose the appropriate abstraction mechanism for a design
- To understand when to use an abstract class and when to use an interface
- To leverage the strengths of each

### Syntax Rules and Structure

**Comparison Table:**

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| **Instantiation** | Cannot instantiate | Cannot instantiate |
| **Fields** | Any fields (instance or static) | Constants only (`public static final`) |
| **Methods** | Abstract or concrete, any access modifier | Abstract, default, static, private (Java 9+) |
| **Inheritance** | Single class extension | Multiple interface implementation |
| **Constructors** | Allowed | Not allowed |
| **Access Modifiers** | Any | Implicitly `public` |
| **When to Use** | Closely related classes sharing code | Unrelated classes sharing behaviour |

**Syntax Rules:**

- "Consider using abstract classes if you want to share code among several closely related classes".
- "Consider using interfaces if you expect that unrelated classes would implement your interface".
- "Consider using abstract classes if you expect that classes that extend your abstract class have many common methods or fields, or require access modifiers other than public".
- "Consider using abstract classes if you want to declare non-static or non-final fields".
- "Consider using interfaces if you want to take advantage of multiple inheritance of type".

**Constraints and Limitations:**

- Abstract classes cannot be multiply inherited.
- Interfaces cannot have instance state.

### Annotated Code Example

```java
// File: AbstractVsInterface.java
// Interface: pure contract
interface Flyable {
    void fly();
}

// Abstract class: partial implementation with state
abstract class Bird {
    protected String name;

    public Bird(String name) {
        this.name = name;
    }

    public abstract void makeSound();

    public void eat() {
        System.out.println(name + " is eating");
    }
}

// Class extends Bird and implements Flyable
class Eagle extends Bird implements Flyable {
    public Eagle(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        System.out.println(name + " screeches");
    }

    @Override
    public void fly() {
        System.out.println(name + " soars through the sky");
    }
}

public class AbstractVsInterface {
    public static void main(String[] args) {
        Eagle eagle = new Eagle("Bald Eagle");
        eagle.eat();        // Bald Eagle is eating
        eagle.makeSound();  // Bald Eagle screeches
        eagle.fly();        // Bald Eagle soars through the sky
    }
}
```

**Expected Output:**
```
Bald Eagle is eating
Bald Eagle screeches
Bald Eagle soars through the sky
```

**Step-by-Step Explanation:**

1. `Flyable` is an interface with one abstract method.
2. `Bird` is an abstract class with a field, constructor, abstract method, and concrete method.
3. `Eagle` extends `Bird` (inheriting state and behaviour) and implements `Flyable` (adding a capability).
4. This demonstrates that a class can extend one abstract class and implement multiple interfaces.

### Real-World Cases

- **`AbstractList` vs `List`:** `List` is an interface; `AbstractList` is an abstract class providing skeletal implementation.
- **`InputStream` vs `Readable`:** `InputStream` is an abstract class; `Readable` is an interface.
- **`AbstractMap` vs `Map`:** `Map` is an interface; `AbstractMap` is an abstract class.

---

## 8. When to Use Abstract Classes

### Definitions

**Core Definition:** Abstract classes should be used when you need to share code and state among closely related classes that form an "is-a" relationship.

**Technical Definition:** "Consider using abstract classes if any of these statements apply to your situation: You want to share code among several closely related classes. You expect that classes that extend your abstract class have many common methods or fields, or require access modifiers other than public (such as protected and private). You want to declare non-static or non-final fields".

**Beginner-Friendly Explanation:** Use an abstract class when you have several classes that are variations of the same thing and share a lot of code and data. For example, `SavingsAccount` and `CheckingAccount` are both `BankAccount` types and share common fields like `balance` and `accountNumber`.

### Purposes

- To share implementation code among closely related classes
- To provide common state through instance fields
- To enforce a common structure while allowing variations
- To support the Template Method pattern

### Syntax Rules and Structure

**Decision Criteria:**

| Criterion | Use Abstract Class |
|-----------|-------------------|
| Classes are closely related | Yes |
| Need to share code | Yes |
| Need instance state (fields) | Yes |
| Need non-public members | Yes |
| Single inheritance is acceptable | Yes |
| Need constructors | Yes |

### Annotated Code Example

```java
// File: WhenToUseAbstract.java
// Abstract class for closely related classes sharing code
abstract class DatabaseConnection {
    protected String url;
    protected String username;
    protected String password;
    protected boolean connected;

    public DatabaseConnection(String url, String username, String password) {
        this.url = url;
        this.username = username;
        this.password = password;
    }

    // Concrete method: shared behaviour
    public void connect() {
        System.out.println("Connecting to " + url + "...");
        connected = true;
        System.out.println("Connected!");
    }

    // Concrete method: shared behaviour
    public void disconnect() {
        connected = false;
        System.out.println("Disconnected from " + url);
    }

    // Abstract method: each database type executes queries differently
    public abstract void executeQuery(String query);
}

class MySQLConnection extends DatabaseConnection {
    public MySQLConnection(String url, String username, String password) {
        super(url, username, password);
    }

    @Override
    public void executeQuery(String query) {
        System.out.println("MySQL executing: " + query);
    }
}

class PostgreSQLConnection extends DatabaseConnection {
    public PostgreSQLConnection(String url, String username, String password) {
        super(url, username, password);
    }

    @Override
    public void executeQuery(String query) {
        System.out.println("PostgreSQL executing: " + query);
    }
}

public class WhenToUseAbstract {
    public static void main(String[] args) {
        DatabaseConnection mysql = new MySQLConnection("jdbc:mysql://localhost/db", "root", "pass");
        DatabaseConnection postgres = new PostgreSQLConnection("jdbc:postgresql://localhost/db", "admin", "pass");

        mysql.connect();                   // Connecting to jdbc:mysql://localhost/db... Connected!
        mysql.executeQuery("SELECT * FROM users");  // MySQL executing: SELECT * FROM users
        mysql.disconnect();                // Disconnected from jdbc:mysql://localhost/db

        postgres.connect();                // Connecting to jdbc:postgresql://localhost/db... Connected!
        postgres.executeQuery("SELECT * FROM orders");  // PostgreSQL executing: SELECT * FROM orders
    }
}
```

**Expected Output:**
```
Connecting to jdbc:mysql://localhost/db...
Connected!
MySQL executing: SELECT * FROM users
Disconnected from jdbc:mysql://localhost/db
Connecting to jdbc:postgresql://localhost/db...
Connected!
PostgreSQL executing: SELECT * FROM orders
```

**Step-by-Step Explanation:**

1. `DatabaseConnection` is an abstract class representing the concept of a database connection.
2. It has shared fields (`url`, `username`, `password`, `connected`), a constructor, and shared concrete methods (`connect`, `disconnect`).
3. The `executeQuery` method is abstract because each database type executes queries differently.
4. `MySQLConnection` and `PostgreSQLConnection` extend the abstract class and implement `executeQuery`.
5. This design shares code and state while allowing database-specific behaviour.

### Real-World Cases

- **JDBC:** `AbstractConnection` could provide common connection logic.
- **Game Development:** `GameCharacter` abstract class with common fields (health, position) and abstract `attack()` method.
- **UI Framework:** `Widget` abstract class with common rendering logic and abstract `draw()` method.

---

## 9. When to Use Interfaces

### Definitions

**Core Definition:** Interfaces should be used when you need to define a contract that unrelated classes can implement, or when you need multiple inheritance of type.

**Technical Definition:** "Consider using interfaces if any of these statements apply to your situation: You expect that unrelated classes would implement your interface. You want to specify the behavior of a particular data type, but not concerned about who implements its behavior. You want to take advantage of multiple inheritance of type".

**Beginner-Friendly Explanation:** Use an interface when you want to say "any class that can do X should implement this interface," regardless of what else that class does. For example, `Comparable` can be implemented by `String`, `Integer`, `Date`, and any custom class.

### Purposes

- To define a contract for unrelated classes
- To enable multiple inheritance of type
- To decouple specification from implementation
- To support functional programming with lambdas

### Syntax Rules and Structure

**Decision Criteria:**

| Criterion | Use Interface |
|-----------|--------------|
| Classes are unrelated | Yes |
| Only need to define behaviour | Yes |
| Need multiple inheritance of type | Yes |
| No shared state required | Yes |
| Functional interface (single method) | Yes |
| API evolution with default methods | Yes |

### Annotated Code Example

```java
// File: WhenToUseInterface.java
// Interface for unrelated classes
interface Printable {
    void print();
}

interface Comparable<T> {
    int compareTo(T other);
}

// Unrelated classes implement the same interface
class Document implements Printable {
    private String content;

    public Document(String content) {
        this.content = content;
    }

    @Override
    public void print() {
        System.out.println("Printing document: " + content);
    }
}

class Photo implements Printable {
    private String fileName;

    public Photo(String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void print() {
        System.out.println("Printing photo: " + fileName);
    }
}

class Employee implements Printable, Comparable<Employee> {
    private String name;
    private int id;

    public Employee(String name, int id) {
        this.name = name;
        this.id = id;
    }

    @Override
    public void print() {
        System.out.println("Employee: " + name + " (ID: " + id + ")");
    }

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }
}

public class WhenToUseInterface {
    public static void main(String[] args) {
        Printable[] printables = {
            new Document("Report.pdf"),
            new Photo("vacation.jpg"),
            new Employee("Alice", 101)
        };

        for (Printable p : printables) {
            p.print();
        }
    }
}
```

**Expected Output:**
```
Printing document: Report.pdf
Printing photo: vacation.jpg
Employee: Alice (ID: 101)
```

**Step-by-Step Explanation:**

1. `Printable` is an interface defining a single method `print()`.
2. `Document`, `Photo`, and `Employee` are unrelated classes that all implement `Printable`.
3. `Employee` also implements `Comparable<Employee>`, demonstrating multiple interface implementation.
4. The array of `Printable` references holds objects of different classes, all of which can be printed polymorphically.

### Real-World Cases

- **`Comparable`:** Implemented by `String`, `Integer`, `Date`, and custom classes.
- **`Runnable`:** Implemented by any class that needs to run in a thread.
- **`Serializable`:** Marker interface implemented by many unrelated classes.
- **`List`, `Set`, `Map`:** Interfaces implemented by various collection classes.

---

## 10. Abstract Class Best Practices

### Definitions

**Core Definition:** Abstract class best practices are guidelines for designing and using abstract classes effectively.

**Technical Definition:** Best practices derive from *Effective Java* and the Java Tutorials. Key practices include: designing for inheritance or prohibiting it, documenting the inheritance contract, keeping abstract classes focused, and using the Template Method pattern appropriately.

**Beginner-Friendly Explanation:** Abstract class best practices are like the rules for building a good foundation — make it solid, document it well, and don't make it too complicated.

### Purposes

- To ensure abstract classes are used correctly
- To avoid common pitfalls
- To create maintainable and extensible designs
- To communicate intent to future developers

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Design for inheritance or prohibit it | Document and test subclass behaviour, or make the class `final`. |
| Document the inheritance contract | Specify how methods behave and what subclasses must not do. |
| Keep abstract classes focused | Avoid god objects; each abstract class should have a single responsibility. |
| Provide protected hooks | Allow subclasses to customize behaviour safely. |
| Use the Template Method pattern | Define algorithm skeletons with abstract hooks. |
| Make fields `private` or `protected` | Use `private` for internal state, `protected` for state subclasses need. |
| Validate in constructors | Check arguments and throw exceptions for invalid values. |
| Consider composition over inheritance | Use "has-a" instead of "is-a" when possible. |

**Syntax Rules:**

- Document abstract methods with Javadoc describing what subclasses must do.
- Use `protected` constructors in abstract classes so only subclasses can call them.
- Avoid overusing abstract classes; prefer interfaces for contracts.

**Constraints and Limitations:**

- Abstract classes create tight coupling between base and subclasses.
- Overuse of inheritance can lead to fragile base class problems.

### Annotated Code Example

```java
// File: AbstractBestPractices.java
/**
 * Abstract base class for shapes.
 * <p>
 * Subclasses must implement {@link #area()} and {@link #perimeter()}.
 * The {@link #describe()} method provides a shared description.
 * </p>
 */
public abstract class AbstractBestPractices {
    // Private fields — encapsulated
    private final String name;

    // Protected constructor — only subclasses can call
    protected AbstractBestPractices(String name) {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        this.name = name;
    }

    // Public getter for private field
    public final String getName() {
        return name;
    }

    // Abstract methods: must be implemented by subclasses
    public abstract double area();
    public abstract double perimeter();

    // Concrete method: shared behaviour
    public void describe() {
        System.out.println(name + " — Area: " + area() + ", Perimeter: " + perimeter());
    }

    // Protected hook: subclasses can override for custom validation
    protected void validate() {
        // default: no validation
    }
}

class Square extends AbstractBestPractices {
    private final double side;

    public Square(double side) {
        super("Square");
        if (side <= 0) throw new IllegalArgumentException("Side must be positive");
        this.side = side;
    }

    @Override
    public double area() {
        return side * side;
    }

    @Override
    public double perimeter() {
        return 4 * side;
    }
}

public class AbstractBestPracticesDemo {
    public static void main(String[] args) {
        Square square = new Square(5);
        square.describe();  // Square — Area: 25.0, Perimeter: 20.0
        System.out.println("Name: " + square.getName());  // Name: Square
    }
}
```

**Expected Output:**
```
Square — Area: 25.0, Perimeter: 20.0
Name: Square
```

**Step-by-Step Explanation:**

1. The class is documented with Javadoc explaining what subclasses must implement.
2. Fields are `private` and `final`, ensuring encapsulation and immutability.
3. The constructor is `protected`, so only subclasses can call it, and it validates arguments.
4. Abstract methods `area()` and `perimeter()` define the contract for subclasses.
5. The `describe()` method is concrete and uses the abstract methods.
6. `Square` implements the abstract methods and validates its own arguments.

### Real-World Cases

- **`AbstractList`:** Well-documented, focused abstract class with protected hooks.
- **`InputStream`:** Provides skeletal implementation with abstract `read()` method.
- **`HttpServlet`:** Abstract class with clear contract for servlet lifecycle methods.

---

## References Links

- Abstract Methods and Classes — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html
- Abstract Methods and Classes — Dev.java - https://dev.java/learn/language/oop/inheritance/abstract-classes/
- Java Language Specification, §8.1.1.1: abstract Classes - https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html
- Java Language Specification, §8.4.3.1: abstract Methods - https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html
- Java Language Specification, §8.4.7: Method Body - https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html
- Abstract Keyword in Java — Scaler Topics - https://www.scaler.com/topics/abstract-keyword-in-java/
- Java abstract Keyword — W3Schools - https://coursera.w3schools.com/java/ref_keyword_abstract.asp
- Abstract Classes in Java (Part 2) — GitHub - https://raw.githubusercontent.com/Sanjay-Mathivanan/JAVA_Zero-to-Advanced/de79e9732a3beca3d163d6a21622400be0b2485e/08_Abstract-Features-of-Java/04_Abstract-Classes-Part2.md
- Constructors in Abstract Classes — Stack Overflow - https://browse.library.kiwix.org/
- Effective Java (3rd Edition) by Joshua Bloch — Item 19: Design and document for inheritance or else prohibit it
- SEI CERT Oracle Coding Standard — MET52-J: Do not use the clone method to copy untrusted method parameters - https://wiki.sei.cmu.edu/confluence/display/java/MET52-JG.+Do+not+use+the+clone+method+to+copy+untrusted+method+parameters