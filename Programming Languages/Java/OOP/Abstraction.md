# Java Abstract Classes: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

An **abstract class** in Java is a class declared with the `abstract` keyword that represents a generalized concept which cannot be instantiated on its own. It serves as a base class that defines common structure and partial implementation for a family of related subclasses, while leaving certain behaviors to be implemented by those subclasses.

### Technical Definition

An abstract class is a class that is declared `abstract`—it may or may not include abstract methods. Abstract classes cannot be instantiated, but they can be subclassed . An abstract method is a method declared without an implementation (without braces, followed by a semicolon) . If a class includes abstract methods, the class itself must be declared `abstract` . When an abstract class is subclassed, the subclass usually provides implementations for all abstract methods in its parent class; if it does not, the subclass must also be declared `abstract` . The body of a method must be a semicolon if the method is abstract .

### Beginner-Friendly Explanation

Think of an abstract class like a job description for "Restaurant Employee." You can't hire a generic "Restaurant Employee"—you hire a waiter, a chef, or a cashier. But the job description defines things common to all employees: they must wear a uniform, clock in, and get paid. It also lists tasks that differ by role: "prepare food" (chef), "take orders" (waiter). The abstract class is the job description—it defines the shared parts and leaves the role-specific parts blank for each specific job (subclass) to fill in.

### Key Characteristics

- **Cannot be instantiated**: You cannot create objects directly from an abstract class .
- **May contain abstract and concrete methods**: Abstract classes can mix methods with and without implementations .
- **Can hold instance state**: Unlike interfaces, abstract classes can declare non-static, non-final instance fields .
- **Supports constructors**: Abstract classes can have constructors, which are invoked by subclass constructors.
- **Single inheritance**: A class can extend only one abstract (or concrete) superclass.
- **Partial abstraction**: Abstract classes provide partial implementation, leaving the rest to subclasses .

### Prerequisites

- Basic understanding of classes, fields, and methods.
- Familiarity with inheritance and the `extends` keyword.
- Knowledge of method overriding.

### Related Programming Areas

- **Interfaces**: An alternative to abstract classes for defining contracts.
- **Template Method Pattern**: A behavioral pattern built on abstract classes .
- **Polymorphism**: Abstract classes enable polymorphic behavior through superclass references.
- **Framework Design**: Abstract classes are common in frameworks (e.g., `AbstractList`, `GenericServlet`).

### Core Concepts / Features

1. The `abstract` Keyword
2. Abstract Methods vs. Concrete Methods
3. Abstract Class Design and Partial Abstraction
4. Template-Style Abstractions (Template Method Pattern)
5. Rules Regarding State (Instance Fields)

---

## Core Concept 1: The `abstract` Keyword

### Definitions

**Core Definition**: The `abstract` keyword is a modifier that declares a class or method as incomplete, requiring subclasses to provide the missing implementation.

**Technical Definition**: The `abstract` modifier can be applied to classes and methods. An abstract class is a class that is declared `abstract`—it may or may not include abstract methods . An abstract method is a method declared without an implementation (without braces, followed by a semicolon) . If a class includes abstract methods, the class itself must be declared `abstract` . Abstract classes cannot be instantiated .

**Beginner-Friendly Explanation**: The `abstract` keyword is like a "TODO" sticker on a document. When you mark a class or method as abstract, you're saying "this part isn't finished—someone else needs to complete it." The compiler enforces this by preventing you from creating objects from an abstract class.

### Purposes

- To declare that a class represents an abstract concept that should not be instantiated directly .
- To define a method signature without implementation, forcing subclasses to provide the behavior .
- To create a common base type for a family of related classes.
- To prevent programmers from accidentally creating meaningless instances of a conceptual class.

### Syntax Rules and Structure

**Complete General Syntax (Abstract Class):**

```java
[access_modifier] abstract class ClassName {
    // fields, constructors, concrete methods, abstract methods
}
```

**Complete General Syntax (Abstract Method):**

```java
[access_modifier] abstract ReturnType methodName(parameters);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `abstract` (class) | Modifier indicating the class cannot be instantiated. |
| `abstract` (method) | Modifier indicating the method has no body. |
| `;` | Replaces the method body for abstract methods. |

**Syntax Rules:**

- An abstract method has no body; it ends with a semicolon .
- A class containing at least one abstract method must be declared `abstract` .
- An abstract class can have zero abstract methods (though this is unusual) .
- Abstract methods cannot be `private` (they must be visible to subclasses).
- Abstract classes can have constructors, but they are called only by subclass constructors.

**Constraints and Limitations:**

- Cannot instantiate an abstract class: `new AbstractClass()` causes a compile error .
- Cannot use `abstract` with `final` (they contradict each other).
- Cannot use `abstract` with `static` (static methods cannot be overridden).
- A subclass that does not implement all abstract methods must itself be declared `abstract` .

### Annotated Complete Code Examples

**Example 1: Basic Abstract Class**

```java
/**
 * Demonstrates an abstract class with an abstract method.
 */
abstract class GraphicObject {
    protected int x, y;  // Instance state

    // Concrete method: shared by all subclasses
    public void moveTo(int newX, int newY) {
        this.x = newX;
        this.y = newY;
    }

    // Abstract method: must be implemented by subclasses
    public abstract void draw();
}

class Circle extends GraphicObject {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing Circle at (" + x + "," + y + ") radius=" + radius);
    }
}

class Rectangle extends GraphicObject {
    private double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public void draw() {
        System.out.println("Drawing Rectangle at (" + x + "," + y + ") " + width + "x" + height);
    }
}

public class AbstractDemo {
    public static void main(String[] args) {
        // GraphicObject g = new GraphicObject(); // Compile error: cannot instantiate

        Circle c = new Circle(5);
        c.moveTo(10, 20);  // Inherited concrete method
        c.draw();          // Overridden abstract method

        Rectangle r = new Rectangle(4, 6);
        r.moveTo(5, 5);
        r.draw();

        // Polymorphic usage
        GraphicObject[] shapes = { c, r };
        for (GraphicObject s : shapes) {
            s.draw();  // Dynamic dispatch
        }
    }
}
```

**Expected Output:**

```
Drawing Circle at (10,20) radius=5.0
Drawing Rectangle at (5,5) 4.0x6.0
Drawing Circle at (10,20) radius=5.0
Drawing Rectangle at (5,5) 4.0x6.0
```

**Why This Output Occurs:**
- `GraphicObject` is abstract and cannot be instantiated.
- `moveTo()` is concrete and inherited by both subclasses.
- `draw()` is abstract; each subclass provides its own implementation.
- The polymorphic loop dispatches to the correct `draw()` based on the runtime type.

### Real-World Cases

- **JDK**: `AbstractMap` provides common methods for `HashMap`, `TreeMap`, and `ConcurrentHashMap` .
- **JDK**: `Number` is an abstract class representing the concept of numbers; `Integer` and `Float` provide specific implementations .
- **Drawing applications**: `GraphicObject` with subclasses `Circle`, `Rectangle`, `Line`, `Bezier` .

### References

- Oracle Java Tutorials – Abstract Methods and Classes - https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html 
- Java Language Specification – Classes - https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html 
- IIT Bombay – Writing Abstract Classes and Methods - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/java/javaOO/abstract.html 

---

## Core Concept 2: Abstract Methods vs. Concrete Methods

### Definitions

**Core Definition**: An **abstract method** is a method signature without a body, requiring subclasses to provide an implementation. A **concrete method** is a method with a complete implementation that can be inherited and used directly.

**Technical Definition**: The body of a method must be a semicolon if the method is `abstract` . An abstract method is a method that is declared without an implementation (without braces, and followed by a semicolon) . A concrete method has a block of code that implements it . Abstract classes can contain a mix of abstract and concrete methods .

**Beginner-Friendly Explanation**: An abstract method is like a form with a blank space: "Signature: _____." Every subclass must fill in the blank with its own signature. A concrete method is a form that already has the answer filled in—subclasses can use it as-is or override it if they want something different.

### Purposes

- **Abstract methods**: To define a required behavior that all subclasses must implement, without providing a default implementation.
- **Concrete methods**: To provide shared behavior that all subclasses can inherit and use directly.
- To enforce a contract while allowing flexibility in implementation .
- To avoid code duplication by centralizing common logic in concrete methods.

### Syntax Rules and Structure

**Abstract Method:**

```java
public abstract ReturnType methodName(parameters);
```

**Concrete Method:**

```java
public ReturnType methodName(parameters) {
    // implementation
    return value;
}
```

**Comparison Table:**

| Aspect | Abstract Method | Concrete Method |
|--------|----------------|-----------------|
| Body | None (semicolon only) | Block of code |
| Implementation | Required in subclass | Provided in superclass |
| `abstract` keyword | Required | Not allowed |
| Inheritance | Must be overridden (unless subclass is abstract) | May be used as-is or overridden |
| Purpose | Define required behavior | Share common behavior |

**Syntax Rules:**

- Abstract methods cannot be `private`, `static`, or `final`.
- Concrete methods can be overridden unless marked `final`.
- A class with at least one abstract method must be abstract .
- Concrete methods in an abstract class can call abstract methods (Template Method pattern) .

**Constraints and Limitations:**

- Abstract methods force subclasses to provide implementations; this can be burdensome if the behavior is optional.
- Concrete methods in abstract classes cannot be called on an abstract instance (since none exists).
- Overriding a concrete method is optional; overriding an abstract method is mandatory.

### Annotated Complete Code Examples

**Example 1: Mixing Abstract and Concrete Methods**

```java
/**
 * Demonstrates abstract and concrete methods in an abstract class.
 */
abstract class DataProcessor {
    // Concrete method: common workflow (Template Method)
    public final void process() {
        readData();          // Abstract: must be implemented
        validateData();      // Concrete: shared
        transformData();     // Abstract: must be implemented
        saveData();          // Concrete: shared
    }

    // Abstract methods: required behavior
    protected abstract void readData();
    protected abstract void transformData();

    // Concrete methods: shared behavior
    protected void validateData() {
        System.out.println("Validating data...");
    }

    protected void saveData() {
        System.out.println("Saving processed data.");
    }
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
}

class JSONProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading JSON data");
    }

    @Override
    protected void transformData() {
        System.out.println("Transforming JSON data");
    }
}

public class MixedMethodsDemo {
    public static void main(String[] args) {
        DataProcessor csv = new CSVProcessor();
        csv.process();
        System.out.println("---");
        DataProcessor json = new JSONProcessor();
        json.process();
    }
}
```

**Expected Output:**

```
Reading CSV data
Validating data...
Transforming CSV data
Saving processed data.
---
Reading JSON data
Validating data...
Transforming JSON data
Saving processed data.
```

**Why This Output Occurs:**
- `process()` is a concrete method that defines the algorithm's skeleton .
- `readData()` and `transformData()` are abstract; each subclass provides its own implementation.
- `validateData()` and `saveData()` are concrete; they are shared by all subclasses.
- The template method `process()` is `final` to prevent subclasses from changing the workflow .

### Real-World Cases

- **CSV/JSON data parsers**: The workflow is the same (read, validate, transform, save), but reading and transforming differ .
- **Authentication systems**: Login process is fixed (validate credentials, verify, generate logs), but verification differs by method .
- **Game character behavior**: Movement, attack, and defense sequences are fixed, but implementations vary by character type .

### References

- Java Language Specification – Method Body - https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html 
- GUVI – Template Method Design Pattern - https://www.guvi.in/blog/template-method-design-pattern-in-java/ 
- Centron – Template Method Design Pattern in Java - https://www.centron.de/en/tutorial/template-method-design-pattern-in-java/ 

---

## Core Concept 3: Abstract Class Design and Partial Abstraction

### Definitions

**Core Definition**: **Partial abstraction** means that an abstract class provides a partial implementation, leaving some methods abstract for subclasses to complete.

**Technical Definition**: Unlike interfaces, abstract classes can contain fields that are not static and final, and they can contain implemented methods. Such abstract classes are similar to interfaces, except that they provide a partial implementation, leaving it to subclasses to complete the implementation . If an abstract class contains only abstract method declarations, it should be declared as an interface instead .

**Beginner-Friendly Explanation**: Partial abstraction is like a partially assembled piece of furniture. The manufacturer (abstract class) has already built the frame, attached the legs, and included the screws. But you (the subclass) need to attach the drawer handles and install the shelves. The hard part is done, but your specific model needs some finishing touches.

### Purposes

- To share code among several closely related classes .
- To provide common methods and fields that subclasses can reuse.
- To define a template that subclasses complete.
- To avoid code duplication while still allowing customization.

### Syntax Rules and Structure

**Design Guidelines:**

| Consider Abstract Class When... | Consider Interface When... |
|--------------------------------|---------------------------|
| You want to share code among closely related classes . | Unrelated classes would implement the interface . |
| Subclasses have many common methods or fields . | You want to specify behavior without concern for implementers . |
| You need access modifiers other than public (protected, private) . | You want multiple inheritance of type . |
| You want to declare non-static or non-final fields . | You only need constants (static final fields). |

**Syntax Rules:**

- An abstract class can have zero abstract methods (rare but legal) .
- An abstract class can implement an interface without implementing all methods, leaving them abstract for subclasses .
- Abstract classes can have constructors, which are called by subclass constructors.
- Abstract classes can have static fields and methods .

**Constraints and Limitations:**

- A class can extend only one abstract class (single inheritance).
- Abstract classes create tighter coupling than interfaces.
- If an abstract class contains only abstract methods, consider using an interface instead .

### Annotated Complete Code Examples

**Example 1: Partial Abstraction with Shared State**

```java
/**
 * Demonstrates partial abstraction with shared state.
 */
abstract class BankAccount {
    // Shared state (instance fields)
    protected String accountNumber;
    protected double balance;

    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }

    // Concrete method: shared behavior
    public double getBalance() {
        return balance;
    }

    // Concrete method: shared behavior with invariant check
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balance += amount;
    }

    // Abstract method: requires subclass-specific logic
    public abstract void withdraw(double amount);

    // Abstract method: requires subclass-specific logic
    public abstract double calculateInterest();
}

class SavingsAccount extends BankAccount {
    private double interestRate;

    public SavingsAccount(String accountNumber, double initialBalance, double interestRate) {
        super(accountNumber, initialBalance);
        this.interestRate = interestRate;
    }

    @Override
    public void withdraw(double amount) {
        if (amount > balance) {
            throw new IllegalStateException("Insufficient funds");
        }
        balance -= amount;
    }

    @Override
    public double calculateInterest() {
        return balance * interestRate;
    }
}

class CheckingAccount extends BankAccount {
    private double overdraftLimit;

    public CheckingAccount(String accountNumber, double initialBalance, double overdraftLimit) {
        super(accountNumber, initialBalance);
        this.overdraftLimit = overdraftLimit;
    }

    @Override
    public void withdraw(double amount) {
        if (amount > balance + overdraftLimit) {
            throw new IllegalStateException("Overdraft limit exceeded");
        }
        balance -= amount;
    }

    @Override
    public double calculateInterest() {
        return 0; // Checking accounts don't earn interest
    }
}

public class PartialAbstractionDemo {
    public static void main(String[] args) {
        SavingsAccount savings = new SavingsAccount("SAV-001", 1000, 0.05);
        savings.deposit(500);
        savings.withdraw(200);
        System.out.println("Savings balance: " + savings.getBalance());
        System.out.println("Savings interest: " + savings.calculateInterest());

        CheckingAccount checking = new CheckingAccount("CHK-001", 500, 100);
        checking.withdraw(550); // Allowed with overdraft
        System.out.println("Checking balance: " + checking.getBalance());
    }
}
```

**Expected Output:**

```
Savings balance: 1300.0
Savings interest: 65.0
Checking balance: -50.0
```

**Why This Output Occurs:**
- `BankAccount` provides shared state (`accountNumber`, `balance`) and concrete methods (`deposit`, `getBalance`) .
- `withdraw()` and `calculateInterest()` are abstract because they differ by account type.
- `SavingsAccount` implements withdrawal with a balance check; `CheckingAccount` allows overdraft.
- The shared `deposit()` method is inherited and used by both.

### Real-World Cases

- **`AbstractMap`**: Provides common methods for `HashMap`, `TreeMap`, and `ConcurrentHashMap` .
- **`AbstractList`**: Provides common list operations for `ArrayList` and `LinkedList`.
- **`InputStream`**: Provides common methods for reading bytes from different sources.

### References

- Oracle Java Tutorials – Abstract Methods and Classes - https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html 
- CMU – Abstract Methods, Interfaces and Classes - https://www.cs.cmu.edu/~clo/www/CMU/DataStructures/Lessons/lesson7_2.htm 
- Stack Overflow – Abstract classes can have state - https://stackoverflow.com/revisions/f396cfcf-e6e0-49ea-a2bb-b25fddf46096/view-source 

---

## Core Concept 4: Template-Style Abstractions (Template Method Pattern)

### Definitions

**Core Definition**: The **Template Method Pattern** is a behavioral design pattern that defines the skeleton of an algorithm in an abstract class, deferring some steps to subclasses.

**Technical Definition**: The Template Method pattern defines the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure . The template method should be declared `final` to prevent subclasses from overriding it . The pattern is used to implement the invariant parts of an algorithm once and leave it up to subclasses to implement the behavior that can vary .

**Beginner-Friendly Explanation**: The Template Method is like a recipe with some steps already filled in and others left blank. "Step 1: Preheat oven to 350°F. Step 2: ______. Step 3: Bake for 25 minutes." Every baker (subclass) fills in Step 2 with their own ingredients, but the overall process stays the same.

### Purposes

- To implement the invariant parts of an algorithm once and leave variable parts to subclasses .
- To avoid code duplication by factoring common behavior into a base class .
- To control subclass extensions by defining "hook" operations at specific points .
- To maintain consistent workflow across all subclasses .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public abstract class AbstractClass {
    // Template method: defines the algorithm skeleton
    public final void templateMethod() {
        step1();          // Concrete or abstract
        step2();          // Abstract: subclass implements
        step3();          // Hook: optional override
        step4();          // Concrete: shared
    }

    protected abstract void step2();  // Must be implemented
    protected void step3() { }        // Hook: default no-op
    private void step1() { /* common */ }
    private void step4() { /* common */ }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Template method | `final` method defining the algorithm skeleton. |
| Abstract steps | Methods that subclasses must implement. |
| Concrete steps | Methods with shared implementation. |
| Hooks | Concrete methods with empty/default implementation that subclasses may override . |

**Syntax Rules:**

- The template method should be `final` to prevent overriding .
- Abstract steps are declared `abstract` and implemented by subclasses.
- Hooks are concrete methods with default (often empty) implementations .
- The template method calls the steps in a fixed order.

**Constraints and Limitations:**

- Tight coupling via inheritance: subclasses are strongly related to the parent's structure .
- Difficult to modify the core workflow once defined .
- Only one template method per class (in practice, though multiple are possible).

### Annotated Complete Code Examples

**Example 1: Template Method for House Building**

```java
/**
 * Demonstrates the Template Method pattern.
 */
abstract class HouseTemplate {
    // Template method: final to prevent overriding
    public final void buildHouse() {
        buildFoundation();
        buildPillars();
        buildWalls();
        buildWindows();
        System.out.println("House is built.");
    }

    // Concrete method: shared foundation
    private void buildFoundation() {
        System.out.println("Building foundation with cement, iron rods and sand");
    }

    // Abstract methods: must be implemented by subclasses
    public abstract void buildPillars();
    public abstract void buildWalls();

    // Hook: default implementation, can be overridden
    private void buildWindows() {
        System.out.println("Building Glass Windows");
    }
}

class WoodenHouse extends HouseTemplate {
    @Override
    public void buildPillars() {
        System.out.println("Building Pillars with Wood coating");
    }

    @Override
    public void buildWalls() {
        System.out.println("Building Wooden Walls");
    }
}

class GlassHouse extends HouseTemplate {
    @Override
    public void buildPillars() {
        System.out.println("Building Pillars with glass coating");
    }

    @Override
    public void buildWalls() {
        System.out.println("Building Glass Walls");
    }
}

public class TemplateMethodDemo {
    public static void main(String[] args) {
        HouseTemplate wooden = new WoodenHouse();
        wooden.buildHouse();
        System.out.println("---");
        HouseTemplate glass = new GlassHouse();
        glass.buildHouse();
    }
}
```

**Expected Output:**

```
Building foundation with cement, iron rods and sand
Building Pillars with Wood coating
Building Wooden Walls
Building Glass Windows
House is built.
---
Building foundation with cement, iron rods and sand
Building Pillars with glass coating
Building Glass Walls
Building Glass Windows
House is built.
```

**Why This Output Occurs:**
- `buildHouse()` is the template method (final) that defines the sequence .
- `buildFoundation()` and `buildWindows()` are concrete (shared).
- `buildPillars()` and `buildWalls()` are abstract; each subclass provides its own version.
- The workflow order is fixed, but the details vary by house type.

### Real-World Cases

- **JDK**: All non-abstract methods of `java.io.InputStream`, `OutputStream`, `Reader`, `Writer` .
- **JDK**: Non-abstract methods of `java.util.AbstractList`, `AbstractSet`, `AbstractMap` .
- **Jakarta EE**: `GenericServlet.init(ServletConfig)` calls `init()` which subclasses override .
- **Data parsers**: CSV, JSON, XML parsers share the same workflow .

### References

- GitHub – Template Method Pattern in Java - https://raw.githubusercontent.com/iluwatar/java-design-patterns/d80eca177b2fada0f31a1e3350663096f3076cda/template-method/README.md 
- Centron – Template Method Design Pattern in Java - https://www.centron.de/en/tutorial/template-method-design-pattern-in-java/ 
- GUVI – Template Method Design Pattern - https://www.guvi.in/blog/template-method-design-pattern-in-java/ 

---

## Core Concept 5: Rules Regarding State (Instance Fields)

### Definitions

**Core Definition**: Abstract classes can declare and hold **instance fields** (non-static, non-final state), unlike interfaces which can only define constants.

**Technical Definition**: With abstract classes, you can declare fields that are not static and final, and define public, protected, and private concrete methods . Abstract classes may have state—they can define instance fields and provide methods working with them . This is a key distinction from interfaces, where all fields are automatically `public`, `static`, and `final` .

**Beginner-Friendly Explanation**: An abstract class is like a house with furniture already in it—the furniture (instance fields) belongs to the house and can be moved around (modified). An interface is like a blueprint with only measurements written on it—those numbers (constants) are permanent and belong to everyone, not to any specific house.

### Purposes

- To share mutable state among subclasses.
- To define properties that are common to all subclasses but may have different values for each instance.
- To allow concrete methods in the abstract class to operate on instance state .
- To support the Template Method pattern where shared state is accessed by both concrete and abstract methods.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
abstract class AbstractClass {
    // Instance field (mutable state)
    protected int instanceField;

    // Constant (immutable state)
    public static final int CONSTANT = 100;

    // Constructor initializes state
    public AbstractClass(int value) {
        this.instanceField = value;
    }

    // Concrete method operating on state
    public int getInstanceField() {
        return instanceField;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Instance field | Non-static, mutable state unique to each object. |
| Constant | Static final field shared by all instances. |
| Constructor | Initializes instance fields. |

**Syntax Rules:**

- Abstract classes can have any access modifier for fields (`private`, `protected`, `public`, package-private) .
- Abstract classes can have instance constructors .
- Abstract classes can have static class initializer code .
- Abstract classes can provide implementations for `equals`, `hashCode`, and `toString` .
- Abstract classes can define `final`, `strictfp`, `synchronized`, or `native` methods .

**Constraints and Limitations:**

- A class can extend only one abstract class (single inheritance) .
- Interface fields are only for defining constants; they are implicitly `static final` .
- Default methods in interfaces cannot access instance fields because interfaces cannot define them .

### Annotated Complete Code Examples

**Example 1: Abstract Class with Instance State**

```java
/**
 * Demonstrates instance state in an abstract class.
 */
abstract class Vehicle {
    // Instance state: unique to each vehicle
    protected String make;
    protected String model;
    protected int year;
    protected double fuelLevel;

    // Constructor initializes state
    public Vehicle(String make, String model, int year, double fuelLevel) {
        this.make = make;
        this.model = model;
        this.year = year;
        this.fuelLevel = fuelLevel;
    }

    // Concrete method operating on state
    public void refuel(double amount) {
        if (amount > 0) {
            fuelLevel += amount;
            System.out.println("Refueled " + amount + " gallons. Current: " + fuelLevel);
        }
    }

    // Abstract method: uses state but requires subclass-specific logic
    public abstract double calculateRange();

    // Concrete method using state
    public String getDescription() {
        return year + " " + make + " " + model;
    }
}

class Car extends Vehicle {
    private double mpg;

    public Car(String make, String model, int year, double fuelLevel, double mpg) {
        super(make, model, year, fuelLevel);
        this.mpg = mpg;
    }

    @Override
    public double calculateRange() {
        return fuelLevel * mpg; // Uses inherited fuelLevel
    }
}

class Truck extends Vehicle {
    private double mpg;
    private double cargoCapacity;

    public Truck(String make, String model, int year, double fuelLevel, double mpg, double cargoCapacity) {
        super(make, model, year, fuelLevel);
        this.mpg = mpg;
        this.cargoCapacity = cargoCapacity;
    }

    @Override
    public double calculateRange() {
        return fuelLevel * mpg * 0.8; // Reduced range due to cargo
    }
}

public class StateDemo {
    public static void main(String[] args) {
        Car car = new Car("Toyota", "Camry", 2025, 10, 30);
        car.refuel(5);
        System.out.println(car.getDescription() + " range: " + car.calculateRange() + " miles");

        Truck truck = new Truck("Ford", "F-150", 2025, 20, 20, 2000);
        truck.refuel(10);
        System.out.println(truck.getDescription() + " range: " + truck.calculateRange() + " miles");
    }
}
```

**Expected Output:**

```
Refueled 5.0 gallons. Current: 15.0
2025 Toyota Camry range: 450.0 miles
Refueled 10.0 gallons. Current: 30.0
2025 Ford F-150 range: 480.0 miles
```

**Why This Output Occurs:**
- `Vehicle` declares instance fields (`make`, `model`, `year`, `fuelLevel`) that are shared by all subclasses .
- `refuel()` and `getDescription()` are concrete methods that operate on the shared state.
- `calculateRange()` is abstract; each subclass provides its own formula using the inherited `fuelLevel`.
- The instance state is unique to each object: the car has its own fuel level, the truck has its own.

### Real-World Cases

- **JDK**: `AbstractMap` has instance fields for the key set and values collection .
- **Banking systems**: `BankAccount` abstract class holds `balance` and `accountNumber` instance state.
- **Game entities**: `GameCharacter` abstract class holds `health`, `position`, and `inventory`.

### References

- Oracle Java Tutorials – Abstract Methods and Classes - https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html 
- Stack Overflow – Abstract classes can have state - https://stackoverflow.com/revisions/f396cfcf-e6e0-49ea-a2bb-b25fddf46096/view-source 
- Stack Overflow – Updated for Java 9 - https://stackoverflow.com/revisions/28305336/7 

---

## References

- Oracle Java Tutorials – Abstract Methods and Classes - https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html 
- Java Language Specification – Classes (JLS §8) - https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html 
- Oracle Java Tutorials – When an Abstract Class Implements an Interface - https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html 
- CMU – Template Method vs Strategy vs Observer - http://www.cs.cmu.edu/~charlie/courses/15-214/2014-spring/slides/12-guis.pdf 
- IIT Bombay – Writing Abstract Classes and Methods - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/java/javaOO/abstract.html 
- Centron – Template Method Design Pattern in Java - https://www.centron.de/en/tutorial/template-method-design-pattern-in-java/ 
- Stack Overflow – Abstract classes can have state - https://stackoverflow.com/revisions/f396cfcf-e6e0-49ea-a2bb-b25fddf46096/view-source 
- GitHub – Template Method Pattern in Java - https://raw.githubusercontent.com/iluwatar/java-design-patterns/d80eca177b2fada0f31a1e3350663096f3076cda/template-method/README.md 
- CMU – Abstract Methods, Interfaces and Classes - https://www.cs.cmu.edu/~clo/www/CMU/DataStructures/Lessons/lesson7_2.htm 
- Stack Overflow – Updated for Java 9 - https://stackoverflow.com/revisions/28305336/7 
- GUVI – Template Method Design Pattern - https://www.guvi.in/blog/template-method-design-pattern-in-java/ 
- Cornell CS100J – Abstract classes - https://www.cs.cornell.edu/courses/cs100j/2005fa/handouts/labs/lab08.html 