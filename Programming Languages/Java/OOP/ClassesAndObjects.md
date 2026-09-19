# Java Classes and Objects: A Comprehensive Cheat Sheet

---

## Topic Overview

### Core Definition

**Java Classes and Objects** form the foundational object-oriented programming (OOP) construct in Java. A **class** is a blueprint or template that defines the structure (fields) and behavior (methods) of a category of things. An **object** is a specific instance of a class, created in memory with its own state and the ability to perform the behaviors defined by its class.

### Technical Definition

In Java, a class is a reference type declared with the `class` keyword that encapsulates fields (instance variables), methods, constructors, nested types, and initializer blocks. An object is a runtime entity created via the `new` operator, which allocates heap memory for the object, invokes a constructor, and returns a reference to the newly created instance. Java references are strongly typed, managed by the garbage collector, and default to `null` when uninitialized. Objects are the fundamental units of encapsulation, inheritance, and polymorphism in Java.

### Beginner-Friendly Explanation

Think of a class as a cookie cutter and objects as the cookies. The cookie cutter (class) defines the shape and size (fields and methods), but it's not a cookie itself. Each cookie (object) made from the cutter has its own identity, its own frosting (state), and can be eaten (behavior) independently. You can make many cookies from one cutter, and each cookie can be decorated differently without affecting the others.

### Key Characteristics

- **Blueprint vs. Instance**: A class is a compile-time construct (blueprint); an object is a runtime entity (instance).
- **Encapsulation**: Classes bundle data (fields) and behavior (methods) and can hide internal details via access modifiers.
- **Identity**: Every object has a unique identity (its memory address), distinct from its state.
- **State and Behavior**: Objects have state (values of fields) and behavior (methods that operate on that state).
- **Reference semantics**: Variables of class types hold references to objects, not the objects themselves.
- **Garbage collection**: Objects are automatically reclaimed by the JVM when no references point to them.
- **Default values**: Fields are initialized to type-specific defaults (`0`, `false`, `null`) if not explicitly initialized.

### Prerequisites

- Basic Java syntax (variables, data types, operators).
- Understanding of the `main` method and program entry point.
- Familiarity with primitive types (`int`, `double`, `boolean`, `char`).

### Related Programming Areas

- **Object-Oriented Programming (OOP)**: Encapsulation, inheritance, polymorphism, and abstraction.
- **Design Patterns**: Factory, Singleton, Builder, Prototype — all rely on class and object design.
- **Memory Management**: Heap allocation, stack references, and garbage collection.
- **API Design**: Classes form the public interface of libraries and frameworks.
- **Concurrency**: Objects as shared state require synchronization.

### Core Concepts / Features

1. Class Definition
2. Object Creation
3. Object References
4. Fields
5. Methods
6. Object State
7. Object Behavior

---

## Core Concept 1: Class Definition

### Definitions

**Core Definition**: A class definition is a declaration that specifies the fields, methods, and other members that objects of that class will have.

**Technical Definition**: A class is declared using the `class` keyword followed by an identifier, an optional type parameter section, an optional `extends` clause, an optional `implements` clause, and a body enclosed in braces. The class body contains member declarations: fields, methods, constructors, nested classes, initializer blocks, and static initializer blocks. The class declaration may include modifiers such as `public`, `abstract`, `final`, and `strictfp`.

**Beginner-Friendly Explanation**: A class definition is like writing a recipe. You list the ingredients (fields) and the steps (methods). Once the recipe is written, you can follow it to make as many dishes (objects) as you want.

### Purposes

- To define a reusable template for creating objects with consistent structure and behavior.
- To encapsulate data and the operations that manipulate it into a single unit.
- To serve as the foundation for inheritance, polymorphism, and abstraction.
- To organize code into logical, maintainable units.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[access_modifier] [modifiers] class ClassName [<TypeParameters>] [extends Superclass] [implements Interfaces] {
    // fields
    // constructors
    // methods
    // nested types
    // initializer blocks
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[access_modifier]` | `public`, `protected`, or package-private (no modifier). |
| `[modifiers]` | `abstract`, `final`, `strictfp`, or combinations. |
| `class` | Keyword declaring a class. |
| `ClassName` | Identifier following UpperCamelCase convention. |
| `[<TypeParameters>]` | Optional generic type parameters. |
| `[extends Superclass]` | Optional single superclass (only one allowed). |
| `[implements Interfaces]` | Optional list of interfaces, comma-separated. |
| `{ ... }` | Class body containing members. |

**Syntax Rules:**

- Only one `public` top-level class per `.java` file, and its name must match the file name.
- A class can extend at most one superclass but can implement multiple interfaces.
- If no constructor is declared, the compiler generates a default no-argument constructor.
- Fields and methods can have any of the four access levels: `public`, `protected`, package-private, or `private`.
- Class names should be nouns in UpperCamelCase (e.g., `BankAccount`).

**Constraints and Limitations:**

- `final` classes cannot be subclassed.
- `abstract` classes cannot be instantiated directly.
- A class cannot extend itself (circular inheritance is illegal).
- Top-level classes cannot be declared `private` or `protected`.

### Annotated Complete Code Examples

**Example 1: A Simple Class Definition**

```java
// Definition of the Car class blueprint
class Car {
    // Fields (State of the object)
    private String brand;
    private int speed;

    // Constructor to initialize the object's state
    public Car(String brand, int speed) {
        this.brand = brand; // 'this' differentiates the field from the parameter
        this.speed = speed;
    }

    // Method to simulate accelerating (Behavior)
    public void accelerate(int increment) {
        this.speed += increment;
        System.out.println(brand + " accelerated to " + speed + " km/h.");
    }

    // Getter method to safely access private data
    public String getBrand() {
        return brand;
    }
}
// Main execution class containing the entry pointpublic class BasicCarDemo {
    public static void main(String[] args) {
        // Instantiating a new Car object using the constructor
        Car myCar = new Car("Toyota", 60);

        System.out.println("My current car is a: " + myCar.getBrand());
        
        // Invoking behavior on the object
        myCar.accelerate(20);
    }
}
```

**Expected Output**
```
My current car is a: Toyota
Toyota accelerated to 80 km/h.
```

**Code Breakdown & Logic**
- Encapsulation: The fields brand and speed are marked private to restrict direct external access. They are accessed securely via the getBrand() method or modified safely through accelerate().
- State Mutation: Passing 20 into accelerate() updates the instance variable speed from 60 to 80, changing the internal state of the myCar object.

**Setup and Execution Guide**
To run any of the examples below:
1. Create a file named exactly after the public class (e.g., BasicCarDemo.java).
2. Open your terminal or command prompt and navigate to the folder containing the file.
3. Compile the code using the Java compiler:
```
javac BasicCarDemo.java
```
4. Execute the compiled bytecode:
```
java BasicCarDemo
```

### Real-World Cases

- **Domain modeling**: Classes represent entities like `User`, `Order`, `Product` in business applications.
- **API design**: Public classes form the contract of libraries (e.g., `ArrayList`, `HashMap`).
- **Framework components**: Spring beans, servlets, and controllers are all classes.
- **Data transfer objects (DTOs)**: Simple classes that carry data between layers.

### References

- Oracle Java Tutorials – Classes - https://docs.oracle.com/javase/tutorial/java/javaOO/classes.html
- Java Language Specification – Class Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html
- Dev.java – Classes and Objects - https://dev.java/learn/classes-objects/

---

## Core Concept 2: Object Creation

### Definitions

**Core Definition**: Object creation is the process of instantiating a class to produce a new object in memory, using the `new` operator and a constructor.

**Technical Definition**: The `new` operator allocates heap memory for a new object of the specified class, initializes its fields to their default values, invokes the matching constructor with the provided arguments, and returns a reference to the newly created object. The constructor initializes the object's state according to its parameters and any field initializers or initializer blocks. If a class declares no constructor, the compiler provides a default no-argument constructor.

**Beginner-Friendly Explanation**: Creating an object is like ordering a custom-made item from a catalog. You specify the model (class) and the customizations (constructor arguments), and the factory (JVM) builds it, fills in the defaults, applies your customizations, and hands you a reference (like a receipt) so you can use it later.

### Purposes

- To bring a class to life by creating a usable instance with its own state.
- To initialize an object's fields to meaningful values via constructors.
- To allocate memory for an object and obtain a reference for later use.
- To create multiple independent instances of the same class.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
ClassName variableName = new ClassName(arguments);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ClassName` | The type of the variable and the class being instantiated. |
| `variableName` | The reference variable that holds the new object. |
| `new` | The operator that allocates memory and invokes a constructor. |
| `ClassName(arguments)` | The constructor call with zero or more arguments. |

**Constructor Syntax:**

```java
[access_modifier] ClassName(parameters) {
    // initialization code
}
```

**Syntax Rules:**

- Constructors have the same name as the class and no return type (not even `void`).
- If no constructor is defined, the compiler generates a default no-argument constructor.
- Constructors can be overloaded (multiple constructors with different parameter lists).
- `this(...)` calls another constructor in the same class; `super(...)` calls a superclass constructor.
- `this(...)` or `super(...)` must be the first statement in a constructor.

**Constraints and Limitations:**

- You cannot call a constructor directly; only `new` invokes it.
- Constructors are not inherited; a subclass must define its own or call a superclass constructor.
- If a class has only private constructors, it cannot be instantiated from outside (e.g., Singleton pattern).
- `new` with a type parameter (e.g., `new T()`) is illegal due to type erasure.

### Annotated Complete Code Examples

**Example 1: Multiple Constructors and Object Creation**

```java
/**
 * Demonstrates object creation with overloaded constructors.
 */
public class Rectangle {
    private double width;
    private double height;

    // No-argument constructor: default 1x1 rectangle
    public Rectangle() {
        this(1.0, 1.0); // Delegate to the two-argument constructor
    }

    // One-argument constructor: square
    public Rectangle(double side) {
        this(side, side); // Delegate to the two-argument constructor
    }

    // Two-argument constructor: full initialization
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    public double area() {
        return width * height;
    }

    public double perimeter() {
        return 2 * (width + height);
    }

    @Override
    public String toString() {
        return "Rectangle[" + width + " x " + height + "]";
    }

    public static void main(String[] args) {
        Rectangle r1 = new Rectangle();          // Default 1x1
        Rectangle r2 = new Rectangle(5.0);       // Square 5x5
        Rectangle r3 = new Rectangle(3.0, 4.0);  // Rectangle 3x4

        System.out.println(r1 + " area=" + r1.area() + " perimeter=" + r1.perimeter());
        System.out.println(r2 + " area=" + r2.area() + " perimeter=" + r2.perimeter());
        System.out.println(r3 + " area=" + r3.area() + " perimeter=" + r3.perimeter());
    }
}
```

**Expected Output:**

```
Rectangle[1.0 x 1.0] area=1.0 perimeter=4.0
Rectangle[5.0 x 5.0] area=25.0 perimeter=20.0
Rectangle[3.0 x 4.0] area=12.0 perimeter=14.0
```

**Why This Output Occurs:**
- `new Rectangle()` invokes the no-argument constructor, which delegates to `this(1.0, 1.0)`, creating a 1×1 rectangle.
- `new Rectangle(5.0)` invokes the one-argument constructor, which delegates to `this(5.0, 5.0)`, creating a 5×5 square.
- `new Rectangle(3.0, 4.0)` invokes the two-argument constructor, creating a 3×4 rectangle.
- Each object has its own `width` and `height` fields, independent of the others.

**Step-by-Step Setup Guide:**
1. Create `Rectangle.java`.
2. Compile with `javac Rectangle.java`.
3. Run with `java Rectangle`.
4. Observe the output.

### Real-World Cases

- **Dependency injection**: Frameworks create objects via constructors and inject dependencies.
- **Factory pattern**: A factory class creates objects with specific configurations.
- **Builder pattern**: A builder creates complex objects step by step.
- **Singleton pattern**: A private constructor ensures only one instance exists.

### References

- Oracle Java Tutorials – Objects - https://docs.oracle.com/javase/tutorial/java/javaOO/objects.html
- Java Language Specification – Class Instance Creation Expressions - https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.9
- Dev.java – Creating Objects - https://dev.java/learn/classes-objects/creating-objects/

---

## Core Concept 3: Object References

### Definitions

**Core Definition**: An object reference is a variable that holds the memory address of an object, allowing access to the object's fields and methods.

**Technical Definition**: In Java, all class-type variables are references. A reference variable stores the address of an object on the heap, not the object itself. Multiple reference variables can point to the same object. When no references point to an object, it becomes eligible for garbage collection. Assigning one reference to another copies the reference, not the object. The `==` operator compares references (identity), while `.equals()` typically compares content (equality), depending on the implementation.

**Beginner-Friendly Explanation**: An object reference is like a remote control. The remote (reference) points to a TV (object). You can have multiple remotes pointing to the same TV. If you throw away all remotes, the TV becomes unreachable and gets recycled (garbage collected). Two different remotes pointing to the same TV are equal in identity, but two remotes pointing to different TVs with the same channel are equal in content.

### Purposes

- To access and manipulate objects without copying their data.
- To allow multiple parts of a program to share the same object.
- To enable polymorphic behavior through reference variables of supertypes.
- To support garbage collection by tracking reachability from references.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
ClassName ref = new ClassName();   // Declaration and initialization
ClassName ref2 = ref;              // Copy the reference (both point to same object)
ref = null;                        // Remove the reference
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ClassName ref` | Declares a reference variable of the given class type. |
| `= new ClassName()` | Creates an object and assigns its reference. |
| `= ref2` | Copies the reference (aliasing). |
| `= null` | Sets the reference to point to no object. |

**Comparison Operators:**

| Operator | Description |
|----------|-------------|
| `==` | Compares references (identity). True if both point to the same object. |
| `!=` | True if references point to different objects. |
| `.equals(Object)` | Compares content; default implementation is identity. |

**Syntax Rules:**

- Reference variables are stored on the stack (for local variables) or in the heap (for fields).
- Default value of an uninitialized reference field is `null`.
- Dereferencing a `null` reference causes `NullPointerException`.
- Java does not support pointer arithmetic; references cannot be manipulated as integers.

**Constraints and Limitations:**

- References are strongly typed: a `String` reference cannot point to an `Integer` object.
- Autoboxing allows primitive-to-wrapper conversions, but they are distinct types.
- The garbage collector may reclaim an object as soon as no references reach it.
- `==` and `.equals()` can give different results; use `.equals()` for value comparison when appropriate.

### Annotated Complete Code Examples

**Example 1: Reference Semantics and Aliasing**

```java
import java.util.*;

/**
 * Demonstrates reference aliasing and comparison.
 */
public class ReferenceDemo {
    public static void main(String[] args) {
        // Create two lists
        List<String> list1 = new ArrayList<>();
        list1.add("Java");

        // list2 points to the SAME object as list1
        List<String> list2 = list1;

        // list3 points to a DIFFERENT object
        List<String> list3 = new ArrayList<>();
        list3.add("Java");

        // Modify via list2 (affects list1 because they are aliases)
        list2.add("Generics");

        System.out.println("list1: " + list1);
        System.out.println("list2: " + list2);
        System.out.println("list3: " + list3);

        // Identity comparison (==)
        System.out.println("list1 == list2? " + (list1 == list2)); // true
        System.out.println("list1 == list3? " + (list1 == list3)); // false

        // Content comparison (.equals)
        System.out.println("list1.equals(list2)? " + list1.equals(list2)); // true
        System.out.println("list1.equals(list3)? " + list1.equals(list3)); // false

        // Null reference
        list3 = null;
        System.out.println("list3 is null? " + (list3 == null));
        // list3.add("x"); // Would throw NullPointerException
    }
}
```

**Expected Output:**

```
list1: [Java, Generics]
list2: [Java, Generics]
list3: [Java]
list1 == list2? true
list1 == list3? false
list1.equals(list2)? true
list1.equals(list3)? false
list3 is null? true
```

**Why This Output Occurs:**
- `list2 = list1` copies the reference, so both point to the same `ArrayList` object. Adding to `list2` also affects `list1`.
- `list3` is a separate object with its own contents `[Java]`, which differs from `list1`'s `[Java, Generics]`.
- `list1 == list2` is `true` because they point to the same object.
- `list1 == list3` is `false` because they point to different objects.
- `list1.equals(list3)` is `false` because their contents differ.

### Real-World Cases

- **Collections**: Lists, maps, and sets store references to objects, not copies.
- **Dependency sharing**: Multiple services share a single configuration object via references.
- **Observer pattern**: Observers hold references to subjects they listen to.
- **Memory leaks**: Long-lived references to unused objects prevent garbage collection.

### References

- Oracle Java Tutorials – Objects - https://docs.oracle.com/javase/tutorial/java/javaOO/objects.html
- Java Language Specification – Reference Types - https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.3
- Dev.java – Object References - https://dev.java/learn/classes-objects/

---

## Core Concept 4: Fields

### Definitions

**Core Definition**: Fields (also called instance variables or member variables) are variables declared inside a class that represent the state of an object or the class.

**Technical Definition**: A field is a member of a class declared with a type, a name, and optional modifiers (e.g., `private`, `static`, `final`, `volatile`, `transient`). Instance fields belong to each object and have independent values per instance. Static fields belong to the class and are shared across all instances. Final fields can be assigned only once (either at declaration or in a constructor). Fields are initialized to default values (`0`, `false`, `null`, etc.) if not explicitly initialized.

**Beginner-Friendly Explanation**: Fields are the "attributes" of an object—the things that describe it. A `Car` object might have fields like `color`, `speed`, and `fuelLevel`. Each car has its own values for these fields. A static field is like a counter shared by all cars, such as `totalCarsProduced`.

### Purposes

- To store the state of an object persistently across method calls.
- To define the properties that distinguish one object from another.
- To share data across all instances of a class via static fields.
- To declare constants using `static final` fields.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[access_modifier] [static] [final] [volatile] [transient] Type fieldName [= initializer];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[access_modifier]` | `public`, `protected`, package-private, or `private`. |
| `[static]` | Indicates a class-level field shared by all instances. |
| `[final]` | Indicates a constant field (assigned once). |
| `[volatile]` | Ensures visibility across threads. |
| `[transient]` | Excludes the field from serialization. |
| `Type` | The data type of the field. |
| `fieldName` | The identifier (lowerCamelCase convention). |
| `[= initializer]` | Optional initial value. |

**Field Categories:**

| Category | Description | Example |
|----------|-------------|---------|
| Instance field | Per-object state | `private int count;` |
| Static field | Class-level state | `private static int total;` |
| Constant | Immutable class-level value | `public static final double PI = 3.14159;` |

**Syntax Rules:**

- Field names should be lowerCamelCase (e.g., `accountBalance`).
- `final` fields must be assigned exactly once, either at declaration or in every constructor.
- `static final` fields are constants and should be named in `UPPER_SNAKE_CASE`.
- Fields are initialized to defaults: `0` for numeric, `false` for boolean, `null` for references.

**Constraints and Limitations:**

- Fields cannot be declared `abstract`.
- `final` fields cannot be reassigned after initialization.
- Static fields cannot reference instance fields directly (no `this` in static context).
- Overuse of public fields breaks encapsulation; prefer private fields with getters/setters.

### Annotated Complete Code Examples

**Example 1: Instance, Static, and Final Fields**

```java
/**
 * Demonstrates different types of fields.
 */
public class Product {
    // Static constant: shared by all instances, immutable
    public static final String CURRENCY = "USD";

    // Static field: shared counter, mutable
    private static int totalProducts = 0;

    // Instance fields: unique to each object
    private final String id;       // Final: assigned once
    private String name;
    private double price;
    private int stockQuantity;

    // Constructor initializes instance fields
    public Product(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.stockQuantity = 0;
        totalProducts++; // Increment shared counter
    }

    // Instance method modifies instance state
    public void restock(int quantity) {
        if (quantity > 0) {
            stockQuantity += quantity;
        }
    }

    // Static method accesses static field
    public static int getTotalProducts() {
        return totalProducts;
    }

    @Override
    public String toString() {
        return String.format("Product[%s, %s, %.2f %s, stock=%d]",
            id, name, price, CURRENCY, stockQuantity);
    }

    public static void main(String[] args) {
        Product p1 = new Product("P001", "Laptop", 999.99);
        Product p2 = new Product("P002", "Mouse", 19.99);

        p1.restock(10);
        p2.restock(50);

        System.out.println(p1);
        System.out.println(p2);
        System.out.println("Total products created: " + Product.getTotalProducts());
        System.out.println("Currency: " + Product.CURRENCY);
    }
}
```

**Expected Output:**

```
Product[P001, Laptop, 999.99 USD, stock=10]
Product[P002, Mouse, 19.99 USD, stock=50]
Total products created: 2
Currency: USD
```

**Why This Output Occurs:**
- `CURRENCY` is a `static final` constant shared by all instances.
- `totalProducts` is a `static` field incremented in the constructor, so it reaches `2`.
- `id` is a `final` instance field, set once in the constructor and never changed.
- `name`, `price`, and `stockQuantity` are instance fields unique to each object.
- `p1.restock(10)` modifies only `p1`'s `stockQuantity`; `p2.restock(50)` modifies only `p2`'s.

### Real-World Cases

- **Entity modeling**: `Customer` fields include `id`, `name`, `email`, `address`.
- **Configuration**: `static final` fields for application-wide constants.
- **Counters and statistics**: `static` fields tracking instances created or operations performed.
- **Serialization**: `transient` fields excluded from persistence (e.g., cached values).

### References

- Oracle Java Tutorials – Declaring Member Variables - https://docs.oracle.com/javase/tutorial/java/javaOO/variables.html
- Java Language Specification – Field Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3
- Dev.java – Fields - https://dev.java/learn/classes-objects/fields/

---

## Core Concept 5: Methods

### Definitions

**Core Definition**: Methods are named blocks of code declared inside a class that define the behavior of objects and can be invoked to perform operations.

**Technical Definition**: A method is a member of a class declared with a return type, a name, a parameter list, an optional `throws` clause, and a body. Methods may be instance methods (invoked on an object, with access to `this`), static methods (invoked on the class, without access to instance state), or constructors (special methods for object initialization). Methods support overloading (same name, different parameter lists), overriding (subclass redefines superclass method), and varargs (variable-length argument lists).

**Beginner-Friendly Explanation**: Methods are the "actions" an object can perform. A `Car` object might have methods like `start()`, `accelerate()`, and `brake()`. Methods can take inputs (parameters) and return outputs (return values). They define what the object can do, just as fields define what the object is.

### Purposes

- To define the behavior of objects and classes.
- To encapsulate logic and promote code reuse.
- To provide a clear interface for interacting with objects.
- To enable polymorphism through method overriding.
- To support modular, testable code through well-defined method contracts.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[access_modifier] [static] [final] [abstract] [synchronized] ReturnType methodName(ParameterList) [throws ExceptionList] {
    // method body
    [return value;]
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[access_modifier]` | `public`, `protected`, package-private, or `private`. |
| `[static]` | Indicates a class-level method (no `this`). |
| `[final]` | Prevents overriding in subclasses. |
| `[abstract]` | Declares a method without a body (must be overridden). |
| `[synchronized]` | Ensures only one thread executes at a time. |
| `ReturnType` | The type of value returned, or `void`. |
| `methodName` | The identifier (lowerCamelCase convention). |
| `ParameterList` | Comma-separated list of parameter declarations. |
| `[throws ExceptionList]` | Checked exceptions the method may throw. |

**Key Method Features:**

| Feature | Description |
|---------|-------------|
| Overloading | Same name, different parameter lists. |
| Overriding | Subclass redefines a superclass method with `@Override`. |
| Varargs | `Type... name` accepts zero or more arguments. |
| `this` | Refers to the current object in instance methods. |
| `super` | Refers to the superclass's version of an overridden method. |

**Syntax Rules:**

- Method names should be verbs in lowerCamelCase (e.g., `calculateTotal`).
- A method with a non-`void` return type must return a value on all code paths.
- `abstract` methods have no body and must be in `abstract` classes or interfaces.
- `static` methods cannot access instance fields or methods directly.
- Method parameters are passed by value (for primitives) or by reference value (for objects).

**Constraints and Limitations:**

- Cannot overload methods that differ only in return type.
- Cannot override `final`, `static`, or `private` methods.
- Varargs must be the last parameter.
- Overloaded methods should have consistent semantics to avoid confusion.

### Annotated Complete Code Examples

**Example 1: Instance, Static, and Overloaded Methods**

```java
/**
 * Demonstrates different types of methods.
 */
public class Calculator {

    // Instance method: uses instance state
    private double memory = 0.0;

    public void store(double value) {
        this.memory = value;
    }

    public double recall() {
        return memory;
    }

    // Static method: no instance state
    public static int add(int a, int b) {
        return a + b;
    }

    // Overloaded method: same name, different parameters
    public static double add(double a, double b) {
        return a + b;
    }

    // Varargs method: accepts variable number of arguments
    public static int sum(int... numbers) {
        int total = 0;
        for (int n : numbers) {
            total += n;
        }
        return total;
    }

    // Method with return value and throws clause
    public static double divide(double a, double b) {
        if (b == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return a / b;
    }

    public static void main(String[] args) {
        // Instance methods
        Calculator calc = new Calculator();
        calc.store(42.0);
        System.out.println("Memory: " + calc.recall());

        // Static methods
        System.out.println("add(int, int): " + Calculator.add(3, 4));
        System.out.println("add(double, double): " + Calculator.add(3.5, 4.5));
        System.out.println("sum(1,2,3,4,5): " + Calculator.sum(1, 2, 3, 4, 5));
        System.out.println("divide(10, 3): " + Calculator.divide(10, 3));

        // Exception handling
        try {
            Calculator.divide(10, 0);
        } catch (ArithmeticException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Memory: 42.0
add(int, int): 7
add(double, double): 8.0
sum(1,2,3,4,5): 15
divide(10, 3): 3.3333333333333335
Caught: Cannot divide by zero
```

**Why This Output Occurs:**
- `store(42.0)` sets the instance field `memory` to `42.0`; `recall()` returns it.
- `add(3, 4)` calls the `int` overload, returning `7`.
- `add(3.5, 4.5)` calls the `double` overload, returning `8.0`.
- `sum(1, 2, 3, 4, 5)` uses varargs to sum all arguments, returning `15`.
- `divide(10, 3)` returns `3.333...`; `divide(10, 0)` throws `ArithmeticException`.

### Real-World Cases

- **Service layer**: Methods like `createUser()`, `processOrder()`, `calculateTax()`.
- **Utility classes**: Static methods like `Math.max()`, `Collections.sort()`.
- **Polymorphism**: Overridden methods in subclasses (e.g., `toString()`, `equals()`).
- **API design**: Public methods form the contract of a class.

### References

- Oracle Java Tutorials – Defining Methods - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html
- Java Language Specification – Method Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.4
- Dev.java – Methods - https://dev.java/learn/classes-objects/methods/

---

## Core Concept 6: Object State

### Definitions

**Core Definition**: Object state is the collective set of values stored in an object's instance fields at any given moment.

**Technical Definition**: The state of an object is determined by the values of its instance variables (fields). Each object has its own copy of instance fields, so state is per-instance. State can change over time through method invocations, and it can be read through accessor methods (getters) or direct field access (if permitted). Proper encapsulation ensures that state is modified only through well-defined methods that maintain invariants.

**Beginner-Friendly Explanation**: The state of an object is like the current settings on your phone: brightness, volume, Wi-Fi on/off. Two phones of the same model can have different settings. Methods like "increase volume" change the state, and "get volume" reads it.

### Purposes

- To represent the current condition or data of an object.
- To allow objects to behave differently based on their state.
- To support state-dependent logic (e.g., a `BankAccount` can withdraw only if its balance is sufficient).
- To enable objects to be serialized, compared, and copied based on their state.

### Syntax Rules and Structure

**State is represented by instance fields:**

```java
public class ClassName {
    private Type field1;   // part of the state
    private Type field2;   // part of the state
    // ...
}
```

**State can be read and modified via methods:**

```java
public Type getField1() { return field1; }          // read state
public void setField1(Type value) { field1 = value; } // modify state
```

**Syntax Rules:**

- State is stored in instance fields, not in local variables or method parameters.
- Each object has its own copy of instance fields.
- Static fields are not part of object state; they belong to the class.
- Encapsulation (private fields + public methods) protects state integrity.
- Immutable objects have state that cannot change after construction.

**Constraints and Limitations:**

- Direct field access from outside the class breaks encapsulation.
- Mutable state shared across threads requires synchronization.
- `final` fields cannot change after construction, making them part of immutable state.
- State is lost when an object is garbage collected unless persisted externally.

### Annotated Complete Code Examples

**Example 1: State Management with Encapsulation**

```java
/**
 * Demonstrates object state and encapsulation.
 */
public class Thermostat {
    // State: current temperature and mode
    private double currentTemperature;
    private String mode; // "HEAT", "COOL", "OFF"
    private final double minTemperature;
    private final double maxTemperature;

    public Thermostat(double min, double max) {
        this.minTemperature = min;
        this.maxTemperature = max;
        this.currentTemperature = 22.0; // default
        this.mode = "OFF";
    }

    // Method that modifies state, maintaining invariants
    public void setTemperature(double temperature) {
        if (temperature < minTemperature || temperature > maxTemperature) {
            throw new IllegalArgumentException(
                "Temperature must be between " + minTemperature + " and " + maxTemperature);
        }
        this.currentTemperature = temperature;
    }

    public void setMode(String mode) {
        if (!mode.equals("HEAT") && !mode.equals("COOL") && !mode.equals("OFF")) {
            throw new IllegalArgumentException("Invalid mode: " + mode);
        }
        this.mode = mode;
    }

    // Methods that read state
    public double getCurrentTemperature() { return currentTemperature; }
    public String getMode() { return mode; }

    @Override
    public String toString() {
        return String.format("Thermostat[temp=%.1f, mode=%s]", currentTemperature, mode);
    }

    public static void main(String[] args) {
        Thermostat t = new Thermostat(10.0, 30.0);
        System.out.println("Initial: " + t);

        t.setTemperature(25.0);
        t.setMode("COOL");
        System.out.println("After changes: " + t);

        try {
            t.setTemperature(35.0); // Out of range
        } catch (IllegalArgumentException e) {
            System.out.println("Rejected: " + e.getMessage());
        }

        System.out.println("Final state: " + t);
    }
}
```

**Expected Output:**

```
Initial: Thermostat[temp=22.0, mode=OFF]
After changes: Thermostat[temp=25.0, mode=COOL]
Rejected: Temperature must be between 10.0 and 30.0
Final state: Thermostat[temp=25.0, mode=COOL]
```

**Why This Output Occurs:**
- The constructor initializes the object's state (`22.0`, `OFF`).
- `setTemperature(25.0)` and `setMode("COOL")` change the state, and `toString()` reflects the new values.
- `setTemperature(35.0)` throws `IllegalArgumentException` because it violates the invariant (max = 30.0), so the state remains unchanged.
- The final state is `25.0, COOL` because the rejected change did not modify it.

### Real-World Cases

- **E-commerce**: A `ShoppingCart` object's state (items, quantities, total) changes as users add/remove items.
- **Game development**: A `Player` object's state (health, position, inventory) changes during gameplay.
- **Banking**: An `Account` object's state (balance, transactions) changes with deposits and withdrawals.
- **Workflow engines**: A `Task` object's state (PENDING, IN_PROGRESS, COMPLETED) drives workflow logic.

### References

- Oracle Java Tutorials – Objects - https://docs.oracle.com/javase/tutorial/java/javaOO/objects.html
- Java Language Specification – Fields - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3
- Dev.java – Object State - https://dev.java/learn/classes-objects/

---

## Core Concept 7: Object Behavior

### Definitions

**Core Definition**: Object behavior is the set of actions an object can perform, defined by its methods, and how it responds to method invocations based on its state.

**Technical Definition**: Object behavior is realized through instance methods that operate on the object's state. Behavior can be polymorphic (overridden methods in subclasses), state-dependent (methods produce different results based on field values), and compositional (methods call other methods). Encapsulation ensures that behavior is invoked through well-defined interfaces, and inheritance allows behavior to be shared and extended.

**Beginner-Friendly Explanation**: Behavior is what an object *does*. A `Dog` object can `bark()`, `eat()`, and `sleep()`. The behavior may depend on the dog's state (a hungry dog eats faster). Different dogs may behave differently (a puppy's bark sounds different from an adult's), which is polymorphism.

### Purposes

- To define the operations an object can perform on its state and on other objects.
- To encapsulate logic and hide implementation details behind method interfaces.
- To enable polymorphism, allowing objects of different types to respond to the same message.
- To support state-dependent behavior, where the result of a method depends on the object's current state.
- To enable collaboration between objects through method calls.

### Syntax Rules and Structure

**Behavior is defined by methods:**

```java
public class ClassName {
    public ReturnType behaviorName(parameters) {
        // behavior implementation
    }
}
```

**Behavior invocation:**

```java
object.behaviorName(arguments);
```

**Polymorphic behavior:**

```java
// Superclass
public class Animal {
    public void makeSound() { System.out.println("Some sound"); }
}

// Subclass overrides behavior
public class Dog extends Animal {
    @Override
    public void makeSound() { System.out.println("Woof"); }
}
```

**Syntax Rules:**

- Behavior is invoked via `object.method(args)` for instance methods or `ClassName.method(args)` for static methods.
- Overridden methods must have the same signature (name, parameters, return type or covariant return).
- `@Override` annotation is recommended for clarity and compile-time checking.
- Behavior can be state-dependent: methods may branch on field values.
- Methods can call other methods (including `this.method()`).

**Constraints and Limitations:**

- Static methods cannot access instance state, so they cannot implement object-specific behavior.
- Private methods are not visible to subclasses and cannot be overridden.
- Final methods cannot be overridden.
- Behavior that modifies shared mutable state requires synchronization in concurrent contexts.

### Annotated Complete Code Examples

**Example 1: Polymorphic and State-Dependent Behavior**

```java
/**
 * Demonstrates object behavior: polymorphism and state dependence.
 */
abstract class Shape {
    public abstract double area();
    public abstract double perimeter();

    @Override
    public String toString() {
        return getClass().getSimpleName() +
            "[area=" + String.format("%.2f", area()) +
            ", perimeter=" + String.format("%.2f", perimeter()) + "]";
    }
}

class Circle extends Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() { return Math.PI * radius * radius; }

    @Override
    public double perimeter() { return 2 * Math.PI * radius; }
}

class Rectangle extends Shape {
    private double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() { return width * height; }

    @Override
    public double perimeter() { return 2 * (width + height); }
}

public class BehaviorDemo {
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle(5.0),
            new Rectangle(4.0, 6.0),
            new Circle(2.5)
        };

        for (Shape s : shapes) {
            // Polymorphic behavior: same call, different implementations
            System.out.println(s);
        }

        // State-dependent behavior
        Thermostat t = new Thermostat(10, 30);
        t.setMode("HEAT");
        t.setTemperature(18);
        System.out.println("\nThermostat behavior:");
        System.out.println("Mode: " + t.getMode() + ", Temp: " + t.getCurrentTemperature());
    }
}

class Thermostat {
    private double currentTemperature;
    private String mode;
    private final double minTemperature, maxTemperature;

    public Thermostat(double min, double max) {
        this.minTemperature = min;
        this.maxTemperature = max;
        this.currentTemperature = 22.0;
        this.mode = "OFF";
    }

    public void setTemperature(double temperature) {
        if (temperature < minTemperature || temperature > maxTemperature) {
            throw new IllegalArgumentException("Temperature out of range");
        }
        this.currentTemperature = temperature;
    }

    public void setMode(String mode) {
        if (!mode.equals("HEAT") && !mode.equals("COOL") && !mode.equals("OFF")) {
            throw new IllegalArgumentException("Invalid mode");
        }
        this.mode = mode;
    }

    public double getCurrentTemperature() { return currentTemperature; }
    public String getMode() { return mode; }
}
```

**Expected Output:**

```
Circle[area=78.54, perimeter=31.42]
Rectangle[area=24.00, perimeter=20.00]
Circle[area=19.63, perimeter=15.71]

Thermostat behavior:
Mode: HEAT, Temp: 18.0
```

**Why This Output Occurs:**
- `Shape[] shapes` holds different `Shape` subtypes. The `toString()` call invokes the overridden `area()` and `perimeter()` methods of each concrete type (polymorphism).
- `Circle` computes area as πr² and perimeter as 2πr; `Rectangle` computes area as width×height and perimeter as 2(width+height).
- The `Thermostat` object's behavior depends on its state: `setMode("HEAT")` changes the mode, and `setTemperature(18)` changes the temperature.

### Real-World Cases

- **Polymorphism in frameworks**: Spring's `BeanFactory` returns objects that behave according to their concrete type.
- **Strategy pattern**: Different `PaymentStrategy` implementations have different `pay()` behaviors.
- **Game AI**: Different `Enemy` types have different `attack()` and `move()` behaviors.
- **State machines**: An `Order` object's `cancel()` behavior depends on whether it's PENDING or SHIPPED.

### References

- Oracle Java Tutorials – Methods - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html
- Oracle Java Tutorials – Polymorphism - https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html
- Java Language Specification – Method Invocation Expressions - https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.12
- Dev.java – Object Behavior - https://dev.java/learn/classes-objects/

---

## References

- Oracle Java Tutorials – Classes and Objects - https://docs.oracle.com/javase/tutorial/java/javaOO/
- Oracle Java Tutorials – Classes - https://docs.oracle.com/javase/tutorial/java/javaOO/classes.html
- Oracle Java Tutorials – Objects - https://docs.oracle.com/javase/tutorial/java/javaOO/objects.html
- Oracle Java Tutorials – Declaring Member Variables - https://docs.oracle.com/javase/tutorial/java/javaOO/variables.html
- Oracle Java Tutorials – Defining Methods - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html
- Oracle Java Tutorials – Polymorphism - https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html
- Java Language Specification – Class Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html
- Java Language Specification – Class Instance Creation Expressions - https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.9
- Java Language Specification – Field Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3
- Java Language Specification – Method Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.4
- Java Language Specification – Reference Types - https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.3
- Dev.java – Classes and Objects - https://dev.java/learn/classes-objects/
- Dev.java – Creating Objects - https://dev.java/learn/classes-objects/creating-objects/
- Dev.java – Fields - https://dev.java/learn/classes-objects/fields/
- Dev.java – Methods - https://dev.java/learn/classes-objects/methods/