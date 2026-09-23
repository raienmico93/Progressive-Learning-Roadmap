# Java `static` Keyword: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

The **`static` keyword** in Java is a modifier that associates a member (field, method, nested class, or initializer block) with the **class itself** rather than with any specific instance of the class. Static members belong to the class and are shared by all instances .

### Technical Definition

When a member is declared `static`, it exists exactly once per class, regardless of how many instances are created. Static fields (class variables) are stored in a fixed location in memory associated with the class, not with individual objects. Static methods (class methods) can be invoked using the class name without creating an instance. Static members are initialized when the class is first loaded into the JVM. A static context (such as a static method) prohibits references to `this` and direct access to instance members because there is no current instance to which `this` could refer .

### Beginner-Friendly Explanation

Think of a company. Each employee (instance) has their own desk, computer, and personal belongings (instance variables). But the company itself has a shared conference room, a company-wide email address, and a "total number of employees" counter. These shared things belong to the company (the class), not to any single employee. The `static` keyword marks these shared things—they exist once, and everyone can use them without needing their own personal copy.

### Key Characteristics

- **One per class**: Static members exist once per class, not once per object .
- **Class-level access**: Static members are accessed via the class name (e.g., `ClassName.member`).
- **No `this`**: Static methods cannot use `this` because there is no current instance .
- **Shared state**: All instances share the same static field .
- **Load-time initialization**: Static fields and static initializer blocks run when the class is loaded.
- **Utility methods**: Static methods are ideal for operations that don't depend on instance state.

### Prerequisites

- Basic Java syntax (classes, fields, methods, constructors).
- Understanding of instance vs. class members.
- Familiarity with object creation and the `new` operator.

### Related Programming Areas

- **Utility Classes**: `Math`, `Collections`, `Arrays` rely heavily on static methods.
- **Constants**: `static final` fields define class-level constants .
- **Singleton Pattern**: Static fields hold the single instance.
- **Factory Methods**: Static methods create instances (e.g., `LocalDate.now()`).

### Core Concepts / Features

1. Static Fields (Class-Level State)
2. Static Methods (Class-Level Behavior)
3. Static Initialization Blocks vs. Instance Initialization Blocks
4. Instance vs. Class Members
5. Limitations of Static Contexts
6. Static Imports


## Core Concept 1: Static Fields (Class-Level State)

### Definitions

**Core Definition**: A **static field** (also called a class variable) is a field declared with the `static` modifier that is shared by all instances of the class.

**Technical Definition**: Fields with the `static` modifier are called static fields or class variables. They are associated with the class, rather than with any object. Every instance of the class shares a class variable, which is in one fixed location in memory. Any object can change the value of a class variable, but class variables can also be manipulated without creating an instance of the class . Static fields are initialized when the class is loaded, either to their default values or to explicitly declared initial values .

**Beginner-Friendly Explanation**: A static field is like a whiteboard in a shared office. Everyone in the office sees the same whiteboard, and if one person writes on it, everyone sees the change. In contrast, a personal notebook (instance field) is unique to each person.

### Purposes

- To share data across all instances of a class.
- To track class-level state, such as instance counters or global configuration.
- To declare constants using `static final` .
- To provide a single point of access for data that logically belongs to the class as a whole.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[access_modifier] static [final] Type fieldName [= initialValue];
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[access_modifier]` | `public`, `protected`, package-private, or `private`. |
| `static` | Indicates a class-level field. |
| `[final]` | Optional; makes the field a constant. |
| `Type` | The data type of the field. |
| `fieldName` | The identifier of the field. |
| `[= initialValue]` | Optional initial value. |

**Syntax Rules:**

- Static fields are declared with the `static` modifier.
- Static fields are referenced by the class name: `ClassName.fieldName` .
- Referencing a static field via an instance reference (e.g., `myObject.fieldName`) is discouraged because it obscures the fact that it is a class variable .
- `static final` fields are constants and cannot be reassigned.

**Constraints and Limitations:**

- Static fields are not thread-safe unless properly synchronized.
- Overuse of static mutable fields can lead to global state and testing difficulties.
- Static fields live for the entire lifetime of the program (until the JVM exits).

### Annotated Complete Code Examples

**Example 1: Instance Counter with a Static Field**

```java
/**
 * Demonstrates a static field used as an instance counter.
 */
public class Bicycle {
    private int cadence;
    private int gear;
    private int speed;
    private int id;

    // Static field: shared by all Bicycle instances
    private static int numberOfBicycles = 0;

    public Bicycle(int startCadence, int startSpeed, int startGear) {
        gear = startGear;
        cadence = startCadence;
        speed = startSpeed;

        // Increment the shared counter and assign ID
        id = ++numberOfBicycles;
    }

    public int getID() {
        return id;
    }

    // Static method to access the static field
    public static int getNumberOfBicycles() {
        return numberOfBicycles;
    }

    public static void main(String[] args) {
        Bicycle bike1 = new Bicycle(10, 0, 1);
        Bicycle bike2 = new Bicycle(15, 5, 3);

        System.out.println("Bike 1 ID: " + bike1.getID()); // 1
        System.out.println("Bike 2 ID: " + bike2.getID()); // 2
        System.out.println("Total bicycles: " + Bicycle.getNumberOfBicycles()); // 2
    }
}
```

**Expected Output:**

```
Bike 1 ID: 1
Bike 2 ID: 2
Total bicycles: 2
```

**Why This Output Occurs:**
- `numberOfBicycles` is static, so it is shared by all `Bicycle` objects.
- Each constructor increments the shared counter.
- `bike1` gets ID `1`, `bike2` gets ID `2`.
- `Bicycle.getNumberOfBicycles()` accesses the static field via the class name .

**Step-by-Step Setup Guide:**
1. Create `Bicycle.java`.
2. Compile with `javac Bicycle.java`.
3. Run with `java Bicycle`.
4. Observe the output.

### Real-World Cases

- **Instance counters**: Tracking how many objects of a class have been created .
- **Configuration values**: Static fields holding application-wide settings.
- **Constants**: `static final double PI = 3.14159;` .
- **Caches**: Static maps or lists shared across instances.

### References

- Oracle Java Tutorials – Understanding Class Members - https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html 
- Cornell CS 1130 – Static Methods and Variables - https://www.cs.cornell.edu/courses/cs1130/2016sp/web-lectures/module2/part2/static.pdf 


## Core Concept 2: Static Methods (Class-Level Behavior)

### Definitions

**Core Definition**: A **static method** is a method declared with the `static` modifier that belongs to the class rather than to any instance, and can be invoked without creating an object.

**Technical Definition**: Static methods, which have the `static` modifier in their declarations, should be invoked with the class name, without the need for creating an instance of the class. A common use for static methods is to access static fields . Static methods cannot access instance variables or instance methods directly; they must use an object reference. They also cannot use the `this` keyword because there is no instance for `this` to refer to . The `main` method is static because it must be called before any objects are created .

**Beginner-Friendly Explanation**: A static method is like a company's customer service phone number. You don't need to know a specific employee (instance) to call it—you just dial the number (class name). The method does its job without needing personal data from a specific employee.

### Purposes

- To provide utility methods that don't depend on instance state.
- To access and manipulate static fields.
- To create factory methods that return instances.
- To serve as the entry point for Java applications (`main`) .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[access_modifier] static ReturnType methodName(parameters) {
    // method body
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[access_modifier]` | `public`, `protected`, package-private, or `private`. |
| `static` | Indicates a class-level method. |
| `ReturnType` | The type of value returned, or `void`. |
| `methodName` | The method identifier. |
| `(parameters)` | The parameter list. |

**Syntax Rules:**

- Static methods are invoked via the class name: `ClassName.methodName(args)` .
- Static methods can access static fields and other static methods directly.
- Static methods cannot access instance fields or instance methods directly.
- Static methods cannot use `this` or `super` .
- Referencing a static method via an instance reference is discouraged .

**Constraints and Limitations:**

- Static methods cannot be overridden (they are hidden, not overridden).
- Static methods cannot be `abstract` (in classes).
- Static methods cannot access instance state, making them unsuitable for operations that depend on object-specific data.

### Annotated Complete Code Examples

**Example 1: Static Method Accessing Static Field**

```java
/**
 * Demonstrates a static method that accesses a static field.
 */
public class Counter {
    private static int count = 0;

    // Static method: can access static field directly
    public static void increment() {
        count++;
    }

    // Static method: returns the static field value
    public static int getCount() {
        return count;
    }

    public static void main(String[] args) {
        Counter.increment();
        Counter.increment();
        Counter.increment();

        System.out.println("Count: " + Counter.getCount()); // 3
    }
}
```

**Expected Output:**

```
Count: 3
```

**Why This Output Occurs:**
- `Counter.increment()` is called three times via the class name.
- Each call increments the shared static field `count`.
- `Counter.getCount()` returns the final value `3` .

### Real-World Cases

- **Utility classes**: `Math.sqrt()`, `Collections.sort()`, `Arrays.asList()`.
- **Factory methods**: `LocalDate.now()`, `List.of()`, `Optional.empty()`.
- **Entry point**: `public static void main(String[] args)` .
- **Accessors for static fields**: `getNumberOfBicycles()` .

### References

- Oracle Java Tutorials – Understanding Class Members - https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html 
- SDSC – Declarations and Scope (Static Methods) - https://users.sdsc.edu/~phil/cse11/lectures/lecture10/Lecture10.pdf 


## Core Concept 3: Static Initialization Blocks vs. Instance Initialization Blocks

### Definitions

**Core Definition**: A **static initialization block** is a block of code preceded by the `static` keyword that runs once when the class is loaded. An **instance initialization block** is a block of code without the `static` keyword that runs every time an object is created.

**Technical Definition**: A static initialization block is a normal block of code enclosed in braces `{ }` and preceded by the `static` keyword. A class can have any number of static initialization blocks, and they are called in the order they appear in the source code when the class is first loaded . Instance initializer blocks look just like static initializer blocks, but without the `static` keyword. The Java compiler copies initializer blocks into every constructor, so they run after field initializers and before the constructor body . For a new object, Java runs: superclass initialization, then this class's field initializers and instance blocks (in source order), then the constructor body .

**Beginner-Friendly Explanation**: A static block is like a "class opening ceremony"—it happens once when the class is first used, setting up shared resources. An instance block is like a "new employee onboarding"—it happens every time a new object is created, ensuring each object starts with the same basic setup.

### Purposes

- **Static blocks**: To initialize static fields that require complex logic (error handling, loops, multiple statements) .
- **Instance blocks**: To share initialization code across multiple constructors without duplication .
- To initialize fields whose values need more than a single expression.

### Syntax Rules and Structure

**Complete General Syntax (Static Block):**

```java
static {
    // initialization code
}
```

**Complete General Syntax (Instance Block):**

```java
{
    // initialization code
}
```

**Component Breakdown:**

| Block Type | Keyword | Execution |
|------------|---------|-----------|
| Static block | `static` | Once, when class is loaded |
| Instance block | (none) | Every time an object is created |

**Syntax Rules:**

- Static blocks run in source order when the class is loaded .
- Instance blocks are copied into every constructor and run after field initializers, before the constructor body .
- A class can have multiple static and instance blocks.
- Static blocks can only access static members; instance blocks can access both static and instance members.

**Constraints and Limitations:**

- A field used by an instance block must be declared before the block, or it will have its default value .
- Static blocks that depend on other classes can create subtle load-order problems .
- Overuse of initializer blocks can obscure the initialization flow.

### Annotated Complete Code Examples

**Example 1: Static and Instance Initialization Blocks**

```java
/**
 * Demonstrates static and instance initialization blocks.
 */
public class InitializerDemo {
    // Static field
    private static String staticMessage;

    // Instance field
    private String instanceMessage;

    // Static initialization block: runs once when class is loaded
    static {
        staticMessage = "Static block executed";
        System.out.println("Static block: " + staticMessage);
    }

    // Instance initialization block: runs before each constructor
    {
        instanceMessage = "Instance block executed";
        System.out.println("Instance block: " + instanceMessage);
    }

    // Constructor
    public InitializerDemo() {
        System.out.println("Constructor executed");
    }

    public static void main(String[] args) {
        System.out.println("Main started");
        InitializerDemo obj1 = new InitializerDemo();
        System.out.println("---");
        InitializerDemo obj2 = new InitializerDemo();
    }
}
```

**Expected Output:**

```
Static block: Static block executed
Main started
Instance block: Instance block executed
Constructor executed
---
Instance block: Instance block executed
Constructor executed
```

**Why This Output Occurs:**
- The static block runs **once** when the class is first loaded, before `main` executes .
- The instance block runs **every time** an object is created, before the constructor body .
- `obj1` and `obj2` each trigger the instance block and constructor.

### Real-World Cases

- **Static blocks**: Loading database drivers, reading configuration files into static maps, initializing static caches .
- **Instance blocks**: Shared setup logic across overloaded constructors, initializing complex instance fields .

### References

- Oracle Java Tutorials – Initializing Fields - https://docs.oracle.com/javase/tutorial/java/javaOO/initial.html 
- GitHub – Initializer Blocks (Software Engineering Notes) - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/10-initializer-blocks.md 


## Core Concept 4: Instance vs. Class Members

### Definitions

**Core Definition**: **Instance members** belong to each object individually, while **class members** (static members) belong to the class and are shared by all instances.

**Technical Definition**: When a number of objects are created from the same class blueprint, they each have their own distinct copies of instance variables. Instance members have an existence independent of whoever is calling into them, whereas class members have none until someone calling into them gives them one . Class variables are shared by all instances and exist in one fixed location in memory . Static methods can be invoked without creating an instance, while instance methods require an object .

**Beginner-Friendly Explanation**: Instance members are like personal belongings—each person (object) has their own. Class members are like office supplies—shared by everyone in the office (class) and accessible even when no one is at their desk.

### Purposes

- To clearly distinguish between per-object state and per-class state.
- To decide whether a field or method should be shared or individual.
- To understand memory allocation and lifetime differences.
- To apply the guideline: make a member static if its body does not refer to an instance member .

### Syntax Rules and Structure

**Comparison Table:**

| Aspect | Instance Member | Class (Static) Member |
|--------|----------------|----------------------|
| Belongs to | Each object | The class |
| Memory | Per instance | Once per class |
| Access | `object.member` | `ClassName.member` |
| `this` available | Yes | No |
| Initialization | Constructor / instance block | Static block / field initializer |
| Lifetime | Object lifetime | Program lifetime |

**Syntax Rules:**

- Instance methods can access both instance and static members directly .
- Static methods can access only static members directly .
- Static fields are shared; changes by one instance are visible to all .
- Instance fields are unique; changes to one object's field do not affect others.

**Constraints and Limitations:**

- Static methods cannot access instance members without an object reference.
- Static fields are not thread-safe unless synchronized.
- Overuse of static members can lead to global state and testing difficulties.

### Annotated Complete Code Examples

**Example 1: Instance vs. Static Members**

```java
/**
 * Compares instance and static members.
 */
public class MemberComparison {
    // Instance field: unique per object
    private int instanceCount = 0;

    // Static field: shared by all objects
    private static int staticCount = 0;

    // Instance method: modifies instance field
    public void incrementInstance() {
        instanceCount++;
        staticCount++; // Can also modify static field
    }

    // Static method: modifies only static field
    public static void incrementStatic() {
        staticCount++;
        // instanceCount++; // Compile error: no instance reference
    }

    public int getInstanceCount() { return instanceCount; }
    public static int getStaticCount() { return staticCount; }

    public static void main(String[] args) {
        MemberComparison obj1 = new MemberComparison();
        MemberComparison obj2 = new MemberComparison();

        obj1.incrementInstance(); // instanceCount of obj1 = 1
        obj1.incrementInstance(); // instanceCount of obj1 = 2
        obj2.incrementInstance(); // instanceCount of obj2 = 1

        System.out.println("obj1 instance count: " + obj1.getInstanceCount()); // 2
        System.out.println("obj2 instance count: " + obj2.getInstanceCount()); // 1
        System.out.println("Static count: " + MemberComparison.getStaticCount()); // 3
    }
}
```

**Expected Output:**

```
obj1 instance count: 2
obj2 instance count: 1
Static count: 3
```

**Why This Output Occurs:**
- `instanceCount` is unique to each object; `obj1` and `obj2` have separate copies .
- `staticCount` is shared; all increments affect the same variable .
- `obj1.incrementInstance()` called twice → `obj1.instanceCount = 2`.
- `obj2.incrementInstance()` called once → `obj2.instanceCount = 1`.
- `staticCount` is incremented three times total → `3`.

### Real-World Cases

- **Instance fields**: `Person.name`, `BankAccount.balance`, `Order.items`.
- **Static fields**: `Bicycle.numberOfBicycles`, `Math.PI`, `System.out`.
- **Instance methods**: `list.add()`, `account.withdraw()`.
- **Static methods**: `Math.sqrt()`, `Collections.sort()`, `LocalDate.now()`.

### References

- Oracle Java Tutorials – Understanding Class Members - https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html 
- OpenJDK – Valhalla-dev Discussion (Per-instance vs. per-class members) - https://mail.openjdk.org/pipermail/valhalla-dev/2015-August/001293.html 
- Cornell CS 1130 – Static Methods and Variables - https://www.cs.cornell.edu/courses/cs1130/2016sp/web-lectures/module2/part2/static.pdf 


## Core Concept 5: Limitations of Static Contexts

### Definitions

**Core Definition**: A **static context** is any code region where `this` is not defined—such as a static method, static field initializer, or static initializer block. In a static context, references to instance members and `this` are prohibited.

**Technical Definition**: The purpose of a static context is to demarcate code that must not refer explicitly or implicitly to the current instance of the class whose declaration lexically encloses the static context . A static context prohibits: `this` expressions, unqualified references to instance variables, and references to type parameters or local variables declared by enclosing methods or constructors . Class methods cannot access instance variables or instance methods directly—they must use an object reference. Also, class methods cannot use the `this` keyword as there is no instance for `this` to refer to .

**Beginner-Friendly Explanation**: A static method is like a company's reception desk. The receptionist can answer general questions about the company (static members) but doesn't have access to any specific employee's personal files (instance members) because they don't know which employee you're asking about. They can, however, look up an employee's file if you give them the employee's name (object reference).

### Purposes

- To enforce the separation between class-level and instance-level code.
- To prevent accidental access to instance state from static contexts.
- To make the class-level nature of static methods explicit.
- To avoid compile-time errors from referencing non-existent instances.

### Syntax Rules and Structure

**Prohibited in Static Contexts:**

| Construct | Reason |
|-----------|--------|
| `this` | No current instance exists . |
| Instance fields (unqualified) | No instance to reference. |
| Instance methods (unqualified) | No instance to invoke on. |
| `super` | No current instance. |

**Allowed in Static Contexts:**

| Construct | Example |
|-----------|---------|
| Static fields | `staticField` |
| Static methods | `staticMethod()` |
| Local variables | `int x = 5;` |
| Object references | `obj.instanceField` (with an object) |

**Syntax Rules:**

- Static methods can access static members directly.
- To access instance members from a static method, you must have an object reference: `object.instanceField` .
- Static methods cannot use `this` or `super` .
- Static initializer blocks are also static contexts and follow the same rules.

**Constraints and Limitations:**

- Compile-time errors occur if you violate static context rules.
- Static methods cannot be overridden (they are hidden).
- The `main` method is static, so it cannot directly access instance members without creating an object .

### Annotated Complete Code Examples

**Example 1: Static Context Limitations**

```java
/**
 * Demonstrates what is allowed and prohibited in a static context.
 */
public class StaticLimitations {
    // Instance field
    private int instanceValue = 10;

    // Static field
    private static int staticValue = 20;

    // Static method: cannot access instanceValue directly
    public static void staticMethod() {
        System.out.println("Static value: " + staticValue); // OK

        // System.out.println(instanceValue); // Compile error!
        // System.out.println(this.instanceValue); // Compile error!

        // Allowed: access instance member via object reference
        StaticLimitations obj = new StaticLimitations();
        System.out.println("Instance value via object: " + obj.instanceValue);
    }

    // Instance method: can access both
    public void instanceMethod() {
        System.out.println("Instance value: " + instanceValue); // OK
        System.out.println("Static value: " + staticValue); // OK
        System.out.println("This: " + this); // OK
    }

    public static void main(String[] args) {
        staticMethod();
        new StaticLimitations().instanceMethod();
    }
}
```

**Expected Output:**

```
Static value: 20
Instance value via object: 10
Instance value: 10
Static value: 20
This: StaticLimitations@1b6d3586
```

**Why This Output Occurs:**
- `staticMethod()` can access `staticValue` directly but must use an object reference for `instanceValue` .
- `instanceMethod()` can access both instance and static members directly and can use `this`.
- The commented-out lines would cause compile errors because `this` and unqualified `instanceValue` are not available in the static context.

### Real-World Cases

- **`main` method**: Must create an object to access instance members .
- **Utility methods**: Cannot access instance state without an object parameter.
- **Static initializer blocks**: Cannot reference instance fields.

### References

- Oracle Java Tutorials – Understanding Class Members - https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html 
- OpenJDK – Static Context Discussion - https://mail.openjdk.org/pipermail/amber-spec-observers/attachments/20230208/436ad739/attachment.htm 
- SDSC – Declarations and Scope (Static) - https://users.sdsc.edu/~phil/cse11/lectures/lecture10/Lecture10.pdf 


## Core Concept 6: Static Imports

### Definitions

**Core Definition**: A **static import** is an import declaration that allows static members (fields and methods) to be used without class qualification.

**Technical Definition**: The static import construct allows unqualified access to static members without inheriting from the type containing the static members. Instead, the program imports the members, either individually (`import static java.lang.Math.PI;`) or en masse (`import static java.lang.Math.*;`). Once imported, they may be used without qualification: `double r = cos(PI * theta);` . The static import declaration is analogous to the normal import declaration: normal imports allow classes to be used without package qualification; static imports allow static members to be used without class qualification .

**Beginner-Friendly Explanation**: Normally, you have to say `Math.sqrt(16)` every time you want a square root. With a static import, you can write `sqrt(16)` instead—the class name is no longer needed because you've "imported" the method into your file's namespace.

### Purposes

- To reduce boilerplate when frequently using static members from one or two classes.
- To avoid the Constant Interface Antipattern (putting constants in interfaces) .
- To make code more readable by removing repetitive class qualifications.

### Syntax Rules and Structure

**Complete General Syntax (Single Member):**

```java
import static packageName.ClassName.staticMember;
```

**Complete General Syntax (On-Demand):**

```java
import static packageName.ClassName.*;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `import static` | The static import keyword sequence. |
| `packageName.ClassName` | The fully qualified class name. |
| `staticMember` | The specific static field or method name. |
| `*` | Imports all static members. |

**Syntax Rules:**

- Static imports must appear at the top of the file, after the package declaration and before class declarations.
- Individual imports import a specific static member.
- On-demand imports (`.*`) import all static members of the class .
- Once imported, the static member can be used without class qualification .

**Constraints and Limitations:**

- **Use sparingly** . Overuse can make code unreadable because readers cannot tell which class a static member comes from .
- Importing all static members can pollute the namespace and harm readability .
- If you need only one or two members, import them individually .
- Static imports do not shadow other declarations .

### Annotated Complete Code Examples

**Example 1: Static Import of Math Members**

```java
import static java.lang.Math.PI;
import static java.lang.Math.cos;

/**
 * Demonstrates static imports for Math constants and methods.
 */
public class StaticImportDemo {
    public static void main(String[] args) {
        double theta = Math.toRadians(45); // 45 degrees in radians

        // Without static import:
        // double r = Math.cos(Math.PI * theta);

        // With static import:
        double r = cos(PI * theta);

        System.out.println("cos(PI * theta) = " + r);
    }
}
```

**Expected Output:**

```
cos(PI * theta) = 0.7071067811865476
```

**Why This Output Occurs:**
- `import static java.lang.Math.PI;` allows `PI` to be used without `Math.` .
- `import static java.lang.Math.cos;` allows `cos()` to be used without `Math.`.
- The expression `cos(PI * theta)` is equivalent to `Math.cos(Math.PI * theta)`.

### Real-World Cases

- **Testing**: `import static org.junit.jupiter.api.Assertions.*;` for assertion methods.
- **Math-heavy code**: Importing `PI`, `cos`, `sin`, `sqrt` for readability.
- **Constants**: Importing `Integer.MAX_VALUE`, `Long.MIN_VALUE`.

### References

- OpenJDK – Static Import Guide - https://cr.openjdk.org/~naoto/idna2008/docs_www/technotes/guides/language/static-import.html 
- Oracle Java Tutorials – Static Import (Japanese) - https://docs.oracle.com/javase/jp/8/docs/technotes/guides/language/static-import.html 
- Java Language Specification – Static Import Declarations - https://docs.oracle.com/javase/specs/jls/se6/html/packages.html 


## References

- Oracle Java Tutorials – Understanding Class Members - https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html 
- OpenJDK – Valhalla-dev Discussion (Per-instance vs. per-class members) - https://mail.openjdk.org/pipermail/valhalla-dev/2015-August/001293.html 
- OpenJDK – Static Import Guide - https://cr.openjdk.org/~naoto/idna2008/docs_www/technotes/guides/language/static-import.html 
- Oracle Java Tutorials – Initializing Fields - https://docs.oracle.com/javase/tutorial/java/javaOO/initial.html 
- OpenJDK – Static Context Discussion - https://mail.openjdk.org/pipermail/amber-spec-observers/attachments/20230208/436ad739/attachment.htm 
- Java Language Specification – Static Import Declarations - https://docs.oracle.com/javase/specs/jls/se6/html/packages.html 
- Cornell CS 1130 – Static Methods and Variables - https://www.cs.cornell.edu/courses/cs1130/2016sp/web-lectures/module2/part2/static.pdf 
- SDSC – Declarations and Scope (Static) - https://users.sdsc.edu/~phil/cse11/lectures/lecture10/Lecture10.pdf 
- Oracle Java Tutorials – Static Import (Japanese) - https://docs.oracle.com/javase/jp/8/docs/technotes/guides/language/static-import.html 
- GitHub – Initializer Blocks (Software Engineering Notes) - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/10-initializer-blocks.md 