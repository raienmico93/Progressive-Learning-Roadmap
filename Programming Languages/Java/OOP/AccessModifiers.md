# Java Access Modifiers: A Comprehensive Research Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Access modifiers (also called access specifiers) are keywords in the Java programming language that control the visibility and accessibility of classes, interfaces, fields, methods, and constructors from other parts of a program.

**Technical Definition**
According to the Java Language Specification (JLS §6.6), a member (class, interface, field, or method) of a reference type, or a constructor of a class type, is accessible only if the type is accessible and the member or constructor is declared to permit access . The four access levels in Java are `public`, `protected`, package-private (default, no keyword), and `private`. These modifiers determine whether code in other classes, packages, or subclasses can reference the declared entity.

**Beginner-Friendly Explanation**
Access modifiers are like permission settings on a document. Some documents are open to the public (`public`), some are shared only with family members (`protected`), some are shared only within your department (`package-private`), and some are locked in your personal drawer (`private`). Choosing the right modifier controls who can see and use each part of your Java program.

### Key Characteristics

- **Four Levels of Access**: Java provides exactly four access levels, ordered from most open to most restrictive: `public` > `protected` > package-private > `private` .
- **Compile-Time Enforcement**: Access violations are detected by the Java compiler, not at runtime (with limited exceptions involving reflection).
- **Scope-Based**: Access is determined by the lexical location of the code performing the access and the package structure of the declaring type.
- **Inheritance Interaction**: Access modifiers affect what subclasses can inherit and override, and they impose a rule that overriding methods cannot reduce visibility .
- **Encapsulation Mechanism**: Access modifiers are the primary tool for implementing encapsulation—hiding internal implementation details while exposing a controlled public interface.

### Prerequisites

- Basic understanding of Java syntax (classes, methods, fields)
- Familiarity with Java packages and the `package` declaration
- Knowledge of inheritance (`extends` keyword) and object instantiation
- Understanding of compilation units and the `import` statement

### Related Programming Areas

- **Object-Oriented Programming (OOP)**: Encapsulation is one of the four fundamental OOP principles.
- **API Design**: Access modifiers define the public interface of libraries and frameworks.
- **Modular Programming**: Java 9+ modules interact with access control through exported packages.
- **Design Patterns**: Many patterns (Factory, Singleton, Facade) rely on access control.
- **Security**: Access modifiers are not a security feature against malicious code but enforce programmer intent.

### Core Concepts / Features

The four access levels and the visibility rules governing them.

---

## Core Concept 1: `public` Access Modifier

### Definitions

**Core Definition**
`public` is the most permissive access modifier. Members declared `public` are accessible from anywhere in the program, provided the declaring class itself is accessible.

**Technical Definition**
If a class or interface type is declared `public`, it may be accessed by any Java code that can access the package in which it is declared . If a member or constructor is declared `public`, then access is permitted from any code that can access the declaring type .

**Beginner-Friendly Explanation**
`public` means "everyone can see and use this." It is like a public park—anyone can enter. When you mark something as `public`, you are making a promise that this part of your code is stable and intended for external use.

### Purposes

- To define the public API or interface of a class, package, or library that external code is intended to use.
- To expose methods and fields that need to be accessible from other packages and unrelated classes.
- To allow a class to be instantiated or extended from any other package.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// For top-level classes and interfaces
public class ClassName { }
public interface InterfaceName { }

// For members (fields, methods, constructors)
public returnType methodName(parameters) { }
public type fieldName;
public ClassName(parameters) { }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `public` | Keyword that grants unrestricted access |
| `class` / `interface` | Declaration type being made public |
| `returnType` | The type of value a method returns (or `void`) |
| `methodName` | Identifier for the method |
| `parameters` | Comma-separated list of parameter types and names |
| `type` | The data type of a field |
| `fieldName` | Identifier for the field |

**Syntax Rules**

1. A source file may contain at most one `public` top-level class, and its name must match the filename .
2. `public` can be applied to: top-level classes, interfaces, nested classes, methods, fields, and constructors.
3. All members of interfaces are implicitly `public`; explicitly declaring them `public` is redundant but legal .
4. `public` classes are accessible outside their package only if the package itself is observable (and exported, in modular applications) .

**Constraints and Limitations**

- Local variables and formal parameters cannot have access modifiers; they are effectively private to their scope .
- A top-level class cannot be declared `protected` or `private`.
- The `public` modifier does not override module system restrictions; a `public` class in a non-exported package is not accessible outside the module .

### Annotated Code Examples

**Example 1: Public Class and Members**

```java
// File: com/example/shapes/Circle.java
package com.example.shapes;

public class Circle {                    // Public class: accessible everywhere
    public double radius;                // Public field: accessible everywhere

    public Circle(double radius) {       // Public constructor: instantiate anywhere
        this.radius = radius;
    }

    public double calculateArea() {      // Public method: call from anywhere
        return Math.PI * radius * radius;
    }
}
```

**Step-by-Step Setup Guide**

1. Create a directory structure matching the package: `com/example/shapes/`.
2. Save the code above as `Circle.java` inside that directory.
3. Compile: `javac com/example/shapes/Circle.java`
4. Create a separate file in a different package to test access.

```java
// File: com/example/app/Main.java
package com.example.app;

import com.example.shapes.Circle;         // Import public class from another package

public class Main {
    public static void main(String[] args) {
        Circle c = new Circle(5.0);      // Access public constructor
        System.out.println(c.radius);    // Access public field
        System.out.println(c.calculateArea()); // Call public method
    }
}
```

**Expected Output**

```
5.0
78.53981633974483
```

**Why This Output Occurs**

The `Circle` class is `public`, so `Main` (in a different package) can import and instantiate it. The constructor, field, and method are all `public`, permitting direct access from `Main`. The area calculation uses the standard formula πr², producing approximately 78.54 for radius 5.0.

---

**Example 2: Public Interface**

```java
// File: com/example/services/Calculator.java
package com.example.services;

public interface Calculator {            // Public interface: implementable anywhere
    int add(int a, int b);               // Implicitly public
    int subtract(int a, int b);          // Implicitly public
}
```

```java
// File: com/example/impl/BasicCalculator.java
package com.example.impl;

import com.example.services.Calculator;

public class BasicCalculator implements Calculator {
    @Override
    public int add(int a, int b) {       // Must be public (widening implicit public)
        return a + b;
    }

    @Override
    public int subtract(int a, int b) {
        return a - b;
    }
}
```

```java
// File: com/example/app/CalculatorDemo.java
package com.example.app;

import com.example.services.Calculator;
import com.example.impl.BasicCalculator;

public class CalculatorDemo {
    public static void main(String[] args) {
        Calculator calc = new BasicCalculator();
        System.out.println(calc.add(10, 5));      // 15
        System.out.println(calc.subtract(10, 5)); // 5
    }
}
```

**Expected Output**

```
15
5
```

**Why This Output Occurs**

The interface `Calculator` is `public`, making its methods accessible from any package. The implementing class `BasicCalculator` is also `public`, and its overridden methods are `public` (required, since interface methods are implicitly public). `CalculatorDemo` uses the interface type to invoke methods polymorphically.

### Real-World Cases

- **Java Standard Library**: The `java.util.List` interface is `public`, allowing any application to use lists.
- **Spring Framework**: Public controller methods are exposed as HTTP endpoints via annotations.
- **SDK Development**: A database driver exposes a `public` `Connection` interface while hiding implementation details in package-private classes.

---

## Core Concept 2: `private` Access Modifier

### Definitions

**Core Definition**
`private` is the most restrictive access modifier. Members declared `private` are accessible only from within the body of the top-level class that encloses their declaration.

**Technical Definition**
If a member or constructor is declared `private`, then access is permitted if and only if it occurs within the body of the top-level class that encloses the declaration of the member or constructor .

**Beginner-Friendly Explanation**
`private` means "this is my personal secret." It is like a diary locked in your desk—only you can read it. Other classes, even subclasses, cannot see or touch `private` members.

### Purposes

- To hide implementation details that should not be exposed to any external code, including subclasses.
- To enforce encapsulation by preventing direct modification of internal state.
- To implement helper methods that are used only within the declaring class.
- To restrict constructor access for design patterns like Singleton.

### Syntax Rules and Structure

**Complete General Syntax**

```java
private returnType methodName(parameters) { }
private type fieldName;
private ClassName(parameters) { }
private class NestedClassName { }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `private` | Keyword that restricts access to the enclosing top-level class |
| `returnType` | Method return type |
| `type` | Field data type |
| `ClassName` | Constructor name (matches class name) |
| `NestedClassName` | Name of a nested (inner) class |

**Syntax Rules**

1. `private` cannot be applied to top-level classes or interfaces—only nested types .
2. `private` members are not inherited by subclasses in the sense that subclasses cannot access them directly .
3. A method of class C can access `private` members of *any* instance of class C, not just the `this` reference .
4. Private constructors prevent external instantiation and subclassing .

**Constraints and Limitations**

- `private` members are invisible to Javadoc by default.
- Reflection can bypass `private` access, but this is not intended for normal use.
- If all constructors are `private`, the class cannot be instantiated or extended outside itself .

### Annotated Code Examples

**Example 1: Private Fields with Public Accessors**

```java
// File: com/example/account/Account.java
package com.example.account;

public class Account {
    private String accountNumber;        // Private: hidden from all other classes
    private double balance;              // Private: hidden from all other classes

    public Account(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }

    public double getBalance() {         // Public getter
        return balance;
    }

    public void deposit(double amount) { // Public method controls modification
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

```java
// File: com/example/app/BankApp.java
package com.example.app;

import com.example.account.Account;

public class BankApp {
    public static void main(String[] args) {
        Account acc = new Account("ACC-001", 1000.0);
        System.out.println(acc.getBalance()); // 1000.0

        // acc.balance = 5000.0;  // COMPILE ERROR: balance has private access
        acc.deposit(500.0);
        System.out.println(acc.getBalance()); // 1500.0
    }
}
```

**Expected Output**

```
1000.0
1500.0
```

**Why This Output Occurs**

The `balance` field is `private`, so `BankApp` cannot modify it directly. The only way to change the balance is through the `deposit` method, which validates the input. This ensures the balance cannot be set to an invalid value.

---

**Example 2: Private Constructor (Singleton Pattern)**

```java
// File: com/example/config/AppConfig.java
package com.example.config;

public class AppConfig {
    private static AppConfig instance;   // Single instance held statically

    private AppConfig() {                // Private constructor: no external instantiation
        System.out.println("AppConfig created");
    }

    public static AppConfig getInstance() { // Public factory method
        if (instance == null) {
            instance = new AppConfig();
        }
        return instance;
    }

    public void showConfig() {
        System.out.println("Configuration active");
    }
}
```

```java
// File: com/example/app/ConfigDemo.java
package com.example.app;

import com.example.config.AppConfig;

public class ConfigDemo {
    public static void main(String[] args) {
        // AppConfig c = new AppConfig();  // COMPILE ERROR: constructor is private
        AppConfig c1 = AppConfig.getInstance();
        AppConfig c2 = AppConfig.getInstance();
        System.out.println(c1 == c2);      // true: same instance
        c1.showConfig();
    }
}
```

**Expected Output**

```
AppConfig created
true
Configuration active
```

**Why This Output Occurs**

The `private` constructor prevents any class from calling `new AppConfig()`. The static `getInstance()` method creates the single instance on first call and returns the same instance on subsequent calls. The `true` result confirms that `c1` and `c2` reference the same object.

### Real-World Cases

- **JavaBeans**: Properties are stored in `private` fields with public getter/setter methods.
- **Singleton Pattern**: `private` constructors ensure only one instance of a class exists.
- **Utility Classes**: `private` constructors prevent instantiation of classes containing only static methods (e.g., `java.lang.Math` has a private constructor).

---

## Core Concept 3: `protected` Access Modifier

### Definitions

**Core Definition**
`protected` grants access within the same package and to subclasses in other packages, subject to specific rules regarding instance references.

**Technical Definition**
If a member or constructor is declared `protected`, access is permitted when: (1) access occurs from within the package containing the class in which the protected member is declared; or (2) access occurs within a subclass of the class in which the protected member is declared, and the access is correct as described in JLS §6.6.2 .

**Beginner-Friendly Explanation**
`protected` means "this is for my family." It is like a family recipe shared with your children (subclasses) but not with strangers. Classes in the same package are also considered "family" for this purpose.

### Purposes

- To allow subclasses to access and modify inherited state or behavior while preventing access from unrelated classes in other packages.
- To define extension points in a class hierarchy where subclasses can customize behavior.
- To provide access to internal members for classes working together within the same package.

### Syntax Rules and Structure

**Complete General Syntax**

```java
protected returnType methodName(parameters) { }
protected type fieldName;
protected ClassName(parameters) { }
protected class NestedClassName { }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `protected` | Keyword granting package + subclass access |
| `returnType` | Method return type |
| `type` | Field data type |
| `ClassName` | Constructor name |
| `NestedClassName` | Name of a nested class |

**Syntax Rules**

1. A `protected` member is accessible to all classes in the same package, just like package-private .
2. From outside the package, a `protected` member is accessible only through a subclass reference where the accessing code is "responsible for the implementation" of that object .
3. A `protected` constructor can be accessed by subclass constructor invocations (`super(...)`) from any package .
4. Direct instantiation of a `protected` constructor from a non-subclass in another package is not permitted .

**Constraints and Limitations**

- The "responsible for implementation" rule means you cannot access `protected` members of a superclass-typed reference from a subclass in another package .
- `protected` is not available for top-level classes.
- Overriding methods cannot reduce visibility from `protected` to package-private or `private` .

### Annotated Code Examples

**Example 1: Protected Members in Inheritance**

```java
// File: com/example/vehicle/Vehicle.java
package com.example.vehicle;

public class Vehicle {
    protected int speed;                 // Protected: accessible in same package
                                         // and by subclasses anywhere

    protected void accelerate(int amount) {
        speed += amount;
        System.out.println("Vehicle speed: " + speed);
    }
}
```

```java
// File: com/example/car/Car.java
package com.example.car;

import com.example.vehicle.Vehicle;

public class Car extends Vehicle {
    public void drive() {
        accelerate(30);                  // Access protected method from subclass
        speed = 50;                      // Access protected field from subclass
        System.out.println("Car driving at " + speed);
    }
}
```

```java
// File: com/example/app/VehicleDemo.java
package com.example.app;

import com.example.car.Car;

public class VehicleDemo {
    public static void main(String[] args) {
        Car car = new Car();
        car.drive();
        // car.accelerate(10);           // COMPILE ERROR: not a subclass, different package
    }
}
```

**Expected Output**

```
Vehicle speed: 30
Car driving at 50
```

**Why This Output Occurs**

`Car` is a subclass of `Vehicle`, so it can call the `protected` `accelerate` method and access the `protected` `speed` field. `VehicleDemo` is not a subclass and is in a different package, so it cannot call `accelerate` on a `Car` instance.

---

**Example 2: Protected Access Restriction**

```java
// File: points/Point.java
package points;

public class Point {
    protected int x, y;
}
```

```java
// File: threePoint/Point3d.java
package threePoint;

import points.Point;

public class Point3d extends Point {
    protected int z;

    public void delta(Point p) {
        // p.x += this.x;  // COMPILE ERROR: cannot access p.x
        // p.y += this.y;  // COMPILE ERROR: cannot access p.y
        // Reason: p is of type Point, not Point3d.
        // Point3d is not "responsible for the implementation" of a Point.
    }

    public void delta3d(Point3d q) {
        q.x += this.x;                    // LEGAL: q is Point3d, responsible
        q.y += this.y;                    // LEGAL
        q.z += this.z;                    // LEGAL
    }
}
```

**Step-by-Step Explanation**

1. `Point` declares `protected` fields `x` and `y`.
2. `Point3d` extends `Point` and is in a different package.
3. In `delta(Point p)`, the parameter is typed as `Point`. Even though `Point3d` is a subclass, the accessing code (`Point3d`) is not responsible for implementing the specific `Point` object `p`.
4. In `delta3d(Point3d q)`, the parameter is typed as `Point3d`. Now the accessing code is responsible for the implementation of that object, so access is permitted.

**Why This Rule Exists**

The JLS enforces that protected access from outside the package is limited to the "family" relationship. You can access protected members through `this`, through references of the subclass type, but not through arbitrary superclass references .

### Real-World Cases

- **Template Method Pattern**: A `protected` method defines a hook that subclasses can override to customize behavior.
- **Abstract Classes**: Protected abstract methods force subclasses to provide implementations while hiding the method from external callers.
- **Framework Extension**: Spring's `WebSecurityConfigurerAdapter` used protected methods for customization before deprecation.

---

## Core Concept 4: Package-Private (Default) Access

### Definitions

**Core Definition**
Package-private access (also called "default access") is granted when no access modifier keyword is specified. Members with package-private access are visible only within the same package.

**Technical Definition**
If none of the access modifiers `public`, `protected`, or `private` are specified, a class member or constructor is accessible throughout the package that contains the declaration of the class in which the member is declared, but is not accessible in any other package .

**Beginner-Friendly Explanation**
Package-private means "this is for my team." It is like a department memo—everyone in your department (package) can read it, but people outside the department cannot.

### Purposes

- To allow classes within the same package to collaborate closely without exposing internal details to the rest of the world.
- To define helper classes and utility methods that are implementation details of a package.
- To enforce encapsulation at the package level while enabling tight coupling between related classes.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// No keyword is used for package-private access
class ClassName { }
interface InterfaceName { }
returnType methodName(parameters) { }
type fieldName;
ClassName(parameters) { }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| (no keyword) | The absence of an access modifier |
| `class` / `interface` | Declaration type |
| `returnType` | Method return type |
| `type` | Field data type |

**Syntax Rules**

1. Package-private is the default for all declarations that do not specify `public`, `protected`, or `private`.
2. A top-level class with package-private access can only be accessed from within its own package .
3. Package-private members are accessible to all classes in the same package, including subclasses and unrelated classes.
4. Subclasses in other packages cannot inherit package-private members .

**Constraints and Limitations**

- Package-private top-level classes are not visible outside their package, even via `import`.
- If a public class has a package-private method, subclasses in other packages cannot override it (because they cannot see it) .
- Package-private access is not transitive across subpackages; `com.example` and `com.example.util` are distinct packages.

### Annotated Code Examples

**Example 1: Package-Private Class and Members**

```java
// File: com/example/internal/Helper.java
package com.example.internal;

class Helper {                           // Package-private class: only same package
    String formatMessage(String msg) {   // Package-private method
        return "[HELPER] " + msg;
    }
}
```

```java
// File: com/example/internal/Service.java
package com.example.internal;

public class Service {
    public void process() {
        Helper h = new Helper();         // LEGAL: same package
        System.out.println(h.formatMessage("Processing"));
    }
}
```

```java
// File: com/example/app/ExternalApp.java
package com.example.app;

// import com.example.internal.Helper;  // COMPILE ERROR: Helper is not public

public class ExternalApp {
    public static void main(String[] args) {
        // Helper h = new Helper();      // COMPILE ERROR: not accessible
    }
}
```

**Expected Output (when running Service)**

```
[HELPER] Processing
```

**Why This Output Occurs**

`Helper` is package-private, so it can be used freely within `com.example.internal`. `Service` is public, so it can be instantiated from anywhere, but its internal use of `Helper` remains hidden. `ExternalApp` cannot import or use `Helper` because it is in a different package.

---

**Example 2: Package-Private Inheritance Limitation**

```java
// File: packageA/Base.java
package packageA;

public class Base {
    void display() {                     // Package-private method
        System.out.println("Base display");
    }
}
```

```java
// File: packageB/Derived.java
package packageB;

import packageA.Base;

public class Derived extends Base {
    // Cannot override display() because it is not accessible from packageB

    public void show() {
        // display();                    // COMPILE ERROR: not accessible
        System.out.println("Derived show");
    }
}
```

**Why This Compile Error Occurs**

The `display()` method in `Base` has package-private access. Although `Derived` extends `Base`, it is in a different package and therefore cannot see or override `display()`. The method is effectively "sealed" within `packageA` .

### Real-World Cases

- **Internal Utility Classes**: A JSON parsing library might have package-private helper classes that are not part of the public API.
- **Test Fixtures**: Test classes in the same package can access package-private members of production classes for unit testing.
- **Module Internals**: Before Java 9 modules, package-private was the primary way to hide implementation details.

---

## Core Concept 5: Visibility Rules and Interactions

### Definitions

**Core Definition**
Visibility rules determine which code locations can access a given declaration, based on the access modifier and the relationship between the accessing code and the declared entity.

**Technical Definition**
The JLS specifies that a member is accessible only if the declaring type is accessible and the member's access modifier permits access from the accessing context . The visibility hierarchy is strictly ordered: `public` > `protected` > package-private > `private` .

**Beginner-Friendly Explanation**
Visibility rules are the "rules of the house" for who can go where. A public room is open to all guests, a protected room is open to family and staff, a package-private room is open only to staff in that department, and a private room is locked to everyone but the owner.

### Purposes

- To determine compile-time legality of member access expressions.
- To enforce encapsulation boundaries that protect internal implementation details.
- To enable controlled extension and collaboration through inheritance and package organization.
- To support the Open/Closed Principle by allowing classes to be open for extension (`protected`) but closed for modification (`private`).

### Syntax Rules and Structure

**Complete Visibility Matrix**

| Access Modifier | Same Class | Same Package | Subclass (Different Package) | Any Class (Different Package) |
|----------------|------------|--------------|------------------------------|-------------------------------|
| `private` | Yes | No | No | No |
| Package-private | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes* | No |
| `public` | Yes | Yes | Yes | Yes |

*With restrictions on the reference type (see protected rules).

**Syntax Rules for Overriding**

1. An overriding method must have the same or wider access than the overridden method .
2. Legal widening: package-private → `protected` → `public`
3. Illegal narrowing: `public` → `protected` → package-private → `private`
4. `private` methods are not overridden; a subclass method with the same signature is a new, unrelated method.

**Constraints and Limitations**

- **Interface Members**: All methods in an interface are implicitly `public`. Fields are implicitly `public static final`.
- **Enum Constants**: Enum constants are always `public` .
- **Local Variables**: Cannot have access modifiers; scope is determined by block structure.
- **Constructor Access**: Determines who can instantiate a class and how subclasses can call `super()`.

### Annotated Code Examples

**Example 1: Override Visibility Widening**

```java
// File: com/example/vehicle/Vehicle.java
package com.example.vehicle;

public class Vehicle {
    protected void start() {              // Protected access
        System.out.println("Vehicle starting");
    }
}
```

```java
// File: com/example/car/Car.java
package com.example.car;

import com.example.vehicle.Vehicle;

public class Car extends Vehicle {
    @Override
    public void start() {                 // LEGAL: widening protected → public
        System.out.println("Car starting");
    }
}
```

```java
// File: com/example/bike/Bike.java
package com.example.bike;

import com.example.vehicle.Vehicle;

public class Bike extends Vehicle {
    // @Override
    // private void start() { }           // ILLEGAL: narrowing protected → private
}
```

**Expected Output (when running Car)**

```
Car starting
```

**Why This Output Occurs**

`Car` widens `start()` from `protected` to `public`, which is legal. The `Bike` example demonstrates an illegal narrowing attempt that would cause a compile-time error. The rule exists to preserve polymorphism: code using a `Vehicle` reference must be able to call `start()` regardless of the actual subclass.

---

**Example 2: Nested Class Access**

```java
// File: com/example/outer/Outer.java
package com.example.outer;

public class Outer {
    private int secret = 42;

    private class Inner {                 // Private inner class
        public void reveal() {
            System.out.println("Secret: " + secret); // Access outer private
        }
    }

    public void demonstrate() {
        Inner inner = new Inner();        // Outer can create Inner
        inner.reveal();
    }
}
```

```java
// File: com/example/app/NestedDemo.java
package com.example.app;

import com.example.outer.Outer;

public class NestedDemo {
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.demonstrate();
        // Outer.Inner inner = outer.new Inner(); // COMPILE ERROR: private
    }
}
```

**Expected Output**

```
Secret: 42
```

**Why This Output Occurs**

`Inner` is `private`, so it is accessible only within the body of `Outer`. The `demonstrate()` method creates an `Inner` instance and calls `reveal()`, which can access the outer class's `private` `secret` field. External code cannot reference `Inner` at all.

### Real-World Cases

- **Builder Pattern**: A public `Builder` nested class with private fields and a private constructor for the outer class.
- **JUnit Test Classes**: Package-private test classes can access package-private production classes.
- **Java Collections Framework**: `java.util.Collections` has public static factory methods and private constructors.

---

## References

- Java Language Specification, Chapter 6.6: Access Control - https://docs.oracle.com/javase/specs/jls/se23/jls23.pdf
- Oracle Java Tutorials: Controlling Access to Members of a Class - https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html
- Java Platform, Standard Edition: AccessController - https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/security/AccessController.html
- Java Security Architecture: GuardedObject and SignedObject - https://docs.oracle.com/javase/jp/8/docs/technotes/guides/security/spec/security-spec.doc7.html
- Coglio, Alessandro. "Checking Access to Protected Members in the Java Virtual Machine." Journal of Object Technology, October 2005.
- OpenJDK Amber Development Mailing List: Sealed Classes and Private Permits - https://mail.openjdk.org/pipermail/amber-dev/2024-August/008917.html