# Java Enum: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** A Java enum is a special data type that enables a variable to be a set of predefined constants, where each constant is a distinct instance of the enum class.

**Technical Definition:** An enum declaration, defined in the Java Language Specification §8.9, specifies a new enum class, a special kind of class. The direct superclass of a non-generic enum class `E` is `Enum<E>`. An enum class has no instances other than those defined by its enum constants. It is a compile-time error to attempt to explicitly instantiate an enum class. Three mechanisms ensure that no instances of an enum class exist beyond those defined by its enum constants: the `final` `clone` method in `Enum` ensures that enum constants can never be cloned; reflective instantiation of enum classes is prohibited; and special treatment by the serialization mechanism ensures that duplicate instances are never created as a result of deserialization.

**Beginner-Friendly Explanation:** An enum is like a list of fixed choices. For example, if you are writing a program about days of the week, you do not want to allow someone to enter "Funday" as a day. An enum lets you define exactly the allowed values: `MONDAY`, `TUESDAY`, ..., `SUNDAY`. Each of these is a constant, and you cannot create any others. The Java enum is more powerful than enums in many other languages because it is a full class that can have fields, methods, and constructors.

### Key Characteristics

- Java enums are implicitly `final` unless they contain at least one enum constant with a class body, in which case they are effectively sealed. A nested enum class is implicitly `static`. 
- Enums implement `Comparable` and `Serializable`, and their serialization format is designed to accommodate arbitrary changes to the enum's implementation. 
- The `equals()` and `hashCode()` methods need not be overridden because each enum constant is a singleton. 
- Enum constants are implicitly `public static final`. The compiler automatically generates the `values()` and `valueOf(String)` methods. 
- Enum types are inherently thread-safe because their instances are initialized during class loading, which is thread-safe.

### Prerequisites

- Readers should be familiar with Java classes, interfaces, generics, and the basic object-oriented concepts of inheritance and polymorphism. 
- A basic understanding of the `switch` statement and `Comparator` interface is helpful for the sections on switch expressions and `EnumSet`/`EnumMap`.

### Related Programming Areas with Explanation

- Java enums are related to design patterns (particularly the Singleton and State patterns). 
- Concurrent programming (through inherent thread safety), collections (through `EnumSet` and `EnumMap`), and domain modeling (where enums represent finite sets of domain values such as order statuses or user roles). 
- Enums are also closely related to the `java.lang.Enum` class, the common base class of all Java language enumeration classes.

## Core Concepts and Key Features

### 1. Core Syntax and Structure

**Core Definition:** The core syntax of an enum declaration consists of the `enum` keyword, a list of enum constants, and optional class body declarations including fields, constructors, and methods.

**Technical Definition:** An enum declaration has the form `{ClassModifier} enum Identifier [TypeParameters] [Superinterfaces] EnumBody`. The `EnumBody` consists of an optional `EnumConstantList`, an optional trailing comma, and optional `EnumBodyDeclarations`. Each `EnumConstant` has the form `{EnumConstantModifier} Identifier [TypeArguments] [ ( [ArgumentList] ) ] [ClassBody]`.

**Beginner-Friendly Explanation:** Writing an enum is like listing a set of named values. You write `enum Day { MONDAY, TUESDAY, WEDNESDAY }` and Java creates a class with exactly three instances. You can also add extra information to each constant, like giving each day a display name.

#### 1.1 `enum` Keyword and Enum Constants

**Definitions**

- **Core Definition:** The `enum` keyword declares a new enum class, and the enum constants are the named instances of that class.
- **Technical Definition:** It is a compile-time error if an enum declaration has the modifier `abstract` or `final`. An enum declaration is implicitly `final` unless it contains at least one enum constant that has a class body. An enum type has no instances other than those defined by its enum constants.
- **Beginner-Friendly Explanation:** The `enum` keyword tells Java you are creating a list of fixed choices. The constants (like `WINTER`, `SPRING`) are the actual values you can use.

**Purposes**

- To declare a new enum class with a fixed set of named constants.
- To provide type-safe constants that cannot be confused with other types.
- To replace the legacy `int Enum` pattern, which lacks type safety, namespace, and compile-time safety.

**Syntax Structures and Rules**

Complete general syntax:

```java
public enum EnumName {
    CONSTANT1, CONSTANT2, CONSTANT3;
}
```

Component breakdown:

- `public`: optional access modifier.
- `enum`: the keyword that declares an enum.
- `EnumName`: the identifier of the enum type.
- `CONSTANT1, CONSTANT2, ...`: the list of enum constants, which are implicitly `public static final`.
- A trailing comma is optional.

Syntax rules: It is a compile-time error to attempt to explicitly instantiate an enum type (`new EnumName()`). An enum class may be generic and may declare superinterfaces.

Constraints and limitations: An enum cannot extend another class (it implicitly extends `Enum<E>`). An enum can implement interfaces. Enum constants must be declared before any other members in the enum body.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic enum declaration**

```java
// Step 1: Declare a simple enum
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// Step 2: Use the enum
public class DayExample {
    public static void main(String[] args) {
        Day today = Day.MONDAY;
        System.out.println("Today is: " + today);
        System.out.println("Name: " + today.name());
        System.out.println("Ordinal: " + today.ordinal());
    }
}
```

Expected Output:

```
Today is: MONDAY
Name: MONDAY
Ordinal: 0
```

**Why the code produces this result:** `Day.MONDAY` creates a reference to the first enum constant. Its `name()` returns the declared name, and its `ordinal()` returns its position (0 for the first constant).

**Example 2: Enum with fields and constructor**

```java
// Step 1: Declare an enum with fields and a constructor
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS(4.869e+24, 6.0518e6),
    EARTH(5.976e+24, 6.37814e6);

    // Step 2: Instance fields
    private final double mass;
    private final double radius;

    // Step 3: Private constructor (implicitly private)
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }

    // Step 4: Method
    public double surfaceGravity() {
        final double G = 6.67300E-11;
        return G * mass / (radius * radius);
    }
}

// Step 5: Use the enum
public class PlanetExample {
    public static void main(String[] args) {
        for (Planet p : Planet.values()) {
            System.out.printf("%s: gravity = %.3f%n", p, p.surfaceGravity());
        }
    }
}
```

Expected Output:

```
MERCURY: gravity = 3.699
VENUS: gravity = 8.871
EARTH: gravity = 9.802
```

**Why the code produces this result:** Each enum constant is created with the constructor arguments provided. The `surfaceGravity()` method uses the instance fields `mass` and `radius` to compute the surface gravity.

**Real-World Cases with Explanation**

Enums are used to represent fixed sets of domain values such as order statuses (`PENDING`, `SHIPPED`, `DELIVERED`), user roles (`ADMIN`, `USER`, `GUEST`), card suits, and HTTP status codes.

**References Links**

- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html
- Enum Classes (Java Language Specification, Java SE 21) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9

#### 1.2 Enum Fields (Instance Variables)

**Definitions**

- **Core Definition:** Instance variables declared in the enum class body that store data associated with each enum constant.
- **Technical Definition:** The body of an enum declaration may contain constructor and member declarations as well as instance and static initializers. Instance fields are typically `private final` and are initialized via the enum constructor.
- **Beginner-Friendly Explanation:** You can attach extra information to each enum constant, such as a color code or a display name. Each constant stores its own copy of this data.

**Purposes**

- To associate data with each enum constant.
- To encapsulate properties that are unique to each constant.
- To provide a structured way to store configuration or metadata for each constant.

**Syntax Structures and Rules**

Complete general syntax:

```java
public enum EnumName {
    CONSTANT1(value1), CONSTANT2(value2);

    private final Type field;

    EnumName(Type field) {
        this.field = field;
    }
}
```

Component breakdown:

- `private final Type field`: the instance variable.
- `EnumName(Type field)`: the constructor that initializes the field.
- `CONSTANT1(value1)`: the constant with its constructor arguments.

Syntax rules: Instance fields are declared after the enum constants. The constructor must be private or package-private (no access modifier); it is implicitly private. The constructor is called when each enum constant is initialized.

Constraints and limitations: Enum constructors cannot be `public` or `protected`. Fields should be `final` if they are not modified after construction.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Enum with instance fields**

```java
public enum TrafficLight {
    RED("Stop", 30),
    YELLOW("Caution", 5),
    GREEN("Go", 45);

    private final String action;
    private final int duration;

    TrafficLight(String action, int duration) {
        this.action = action;
        this.duration = duration;
    }

    public String getAction() { return action; }
    public int getDuration() { return duration; }

    public static void main(String[] args) {
        for (TrafficLight light : values()) {
            System.out.println(light + ": " + light.getAction() + " for " + light.getDuration() + "s");
        }
    }
}
```

Expected Output:

```
RED: Stop for 30s
YELLOW: Caution for 5s
GREEN: Go for 45s
```

**Real-World Cases with Explanation**

Enums with instance fields are used in e-commerce to associate display names and prices with product statuses, in networking to associate HTTP status codes with messages, and in games to associate damage values with weapon types.

**References Links**

- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html
- Enum Body Declarations (JLS §8.9.2) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9.2

#### 1.3 Enum Constructors

**Definitions**

- **Core Definition:** A special method that initializes the fields of each enum constant.
- **Technical Definition:** An enum constructor is implicitly `private`. It is a compile-time error if an enum constructor has a `public`, `protected`, or `default` access modifier. The constructor is called once for each enum constant during class initialization.
- **Beginner-Friendly Explanation:** The enum constructor is like a setup routine that runs once for each constant. You cannot call it yourself; Java calls it automatically when the enum is loaded.

**Purposes**

- To initialize the instance fields of each enum constant.
- To ensure that each constant is properly configured before use.
- To provide a controlled way to set up enum constant data.

**Syntax Structures and Rules**

Complete general syntax:

```java
public enum EnumName {
    CONSTANT1(arg1), CONSTANT2(arg2);

    private final Type field;

    EnumName(Type arg) {
        this.field = arg;
    }
}
```

Component breakdown:

- `EnumName(Type arg)`: the constructor declaration.
- The constructor has no access modifier (implicitly private).

Syntax rules: Enum constructors cannot be invoked directly from other code. They are called automatically when the enum constants are initialized during class loading.

Constraints and limitations: Enum constructors cannot access non-static fields of the enum class other than those being initialized. They cannot throw checked exceptions unless all enum constants declare the same exception.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Private enum constructor**

```java
public enum Color {
    RED(255, 0, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255);

    private final int r, g, b;

    private Color(int r, int g, int b) {
        this.r = r;
        this.g = g;
        this.b = b;
    }

    public String toHex() {
        return String.format("#%02X%02X%02X", r, g, b);
    }

    public static void main(String[] args) {
        for (Color c : values()) {
            System.out.println(c + " -> " + c.toHex());
        }
    }
}
```

Expected Output:

```
RED -> #FF0000
GREEN -> #00FF00
BLUE -> #0000FF
```

**Real-World Cases with Explanation**

Enum constructors are used in configuration systems where each enum constant needs specific initialization data, such as database connection parameters or protocol settings.

**References Links**

- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html
- Enum Constructors (JLS §8.9.2) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9.2

#### 1.4 Enum Methods (Custom Behavior)

**Definitions**

- **Core Definition:** Methods declared in the enum class body that provide behavior for all enum constants.
- **Technical Definition:** An enum class body may contain method declarations. Methods can be concrete or abstract (if each constant provides an implementation).
- **Beginner-Friendly Explanation:** You can add methods to an enum just like you add methods to a regular class. All constants share these methods.

**Purposes**

- To provide behavior associated with the enum type.
- To encapsulate logic that operates on the enum constant's data.
- To support polymorphic behavior when combined with constant-specific class bodies.

**Syntax Structures and Rules**

Complete general syntax:

```java
public enum EnumName {
    CONSTANT1, CONSTANT2;

    public ReturnType methodName() {
        // implementation
    }
}
```

Component breakdown:

- `public ReturnType methodName()`: the method declaration.
- The method can access the enum's instance fields.

Syntax rules: Methods are declared after the enum constants and any fields. Static methods can be declared as well.

Constraints and limitations: Enum methods are subject to the usual rules of Java methods.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Enum with custom method**

```java
public enum Operation {
    PLUS {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE {
        public double apply(double x, double y) { return x / y; }
    };

    public abstract double apply(double x, double y);

    public static void main(String[] args) {
        for (Operation op : values()) {
            System.out.println("10 " + op + " 5 = " + op.apply(10, 5));
        }
    }
}
```

Expected Output:

```
10 PLUS 5 = 15.0
10 MINUS 5 = 5.0
10 TIMES 5 = 50.0
10 DIVIDE 5 = 2.0
```

**Real-World Cases with Explanation**

Enums with custom methods are used in calculators (as shown above), in state machines where each state has behavior, and in strategy patterns where each constant implements a different algorithm.

**References Links**

- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html
- Enum Body Declarations (JLS §8.9.2) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9.2

### 2. Built-in Methods and Properties

**Core Definition:** The `java.lang.Enum` class provides several built-in methods that all enum types inherit, and the compiler generates additional static methods for each enum type.

**Technical Definition:** The `Enum<E>` class declares `name()`, `ordinal()`, `compareTo()`, `equals()`, `hashCode()`, `toString()`, and `clone()`. The compiler additionally generates `values()` and `valueOf(String)` as static methods for each enum type.

**Beginner-Friendly Explanation:** Every enum comes with a set of ready-made methods. You can get the name of a constant, its position, compare two constants, and look up a constant by name.

#### 2.1 `values()` Method

**Definitions**

- **Core Definition:** A static method generated by the compiler that returns an array of all enum constants in declaration order.
- **Technical Definition:** `public static E[] values()` returns an array containing all the constants of this enum type, in the order they are declared. This method is synthesized by the compiler and is not part of the `Enum` class.
- **Beginner-Friendly Explanation:** `values()` gives you an array of all the constants in the order you declared them. It is useful for looping through all enum values.

**Purposes**

- To iterate over all enum constants.
- To obtain an array of constants for processing.
- To support dynamic operations that require the full set of constants.

**Syntax Structures and Rules**

Complete general syntax:

```java
EnumType[] constants = EnumType.values();
```

Component breakdown:

- `EnumType`: the enum type.
- Returns an array of all enum constants.

Syntax rules: The array is a fresh copy each time; modifying it does not affect the enum. The order is the declaration order.

Constraints and limitations: The returned array is a copy; changes to it do not affect the enum. The method is static and implicitly generated.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Iterating with `values()`**

```java
public enum Season { WINTER, SPRING, SUMMER, FALL }

public class ValuesExample {
    public static void main(String[] args) {
        Season[] seasons = Season.values();
        for (int i = 0; i < seasons.length; i++) {
            System.out.println((i + 1) + ". " + seasons[i]);
        }
    }
}
```

Expected Output:

```
1. WINTER
2. SPRING
3. SUMMER
4. FALL
```

**Real-World Cases with Explanation**

`values()` is used in dropdown menus where all enum values must be displayed, in configuration parsing where all options must be validated, and in tests that iterate over all possible enum constants.

**References Links**

- Enum (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html
- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html

#### 2.2 `valueOf(String)` Method

**Definitions**

- **Core Definition:** A static method generated by the compiler that returns the enum constant with the specified name.
- **Technical Definition:** `public static E valueOf(String name)` returns the enum constant of this type with the specified name. The string must match exactly an identifier used to declare an enum constant in this type. (Extraneous whitespace characters are not permitted.) Throws `IllegalArgumentException` if the specified enum type has no constant with the specified name, or `NullPointerException` if the argument is `null`.
- **Beginner-Friendly Explanation:** `valueOf("MONDAY")` gives you `Day.MONDAY`. It throws an exception if you pass a name that does not exist.

**Purposes**

- To convert a string name into the corresponding enum constant.
- To support parsing of user input or configuration data.
- To provide a type-safe lookup mechanism.

**Syntax Structures and Rules**

Complete general syntax:

```java
EnumType constant = EnumType.valueOf("CONSTANT_NAME");
```

Component breakdown:

- `"CONSTANT_NAME"`: the exact name of the enum constant.
- Returns the enum constant.

Syntax rules: The name must match exactly, including case. Leading or trailing whitespace causes an exception.

Constraints and limitations: Throws `IllegalArgumentException` if the name is invalid. Throws `NullPointerException` if the name is `null`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `valueOf` lookup**

```java
public enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY }

public class ValueOfExample {
    public static void main(String[] args) {
        Day day = Day.valueOf("WEDNESDAY");
        System.out.println("Day: " + day);

        try {
            Day invalid = Day.valueOf("FUNDAY");
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Day: WEDNESDAY
Caught: IllegalArgumentException
```

**Real-World Cases with Explanation**

`valueOf` is used in command-line argument parsing, in deserialization of configuration files, and in REST API controllers where enum values are passed as strings.

**References Links**

- Enum (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html
- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html

#### 2.3 `ordinal()` Method

**Definitions**

- **Core Definition:** Returns the position of the enum constant in its declaration, starting from zero.
- **Technical Definition:** `public final int ordinal()` returns the ordinal of this enumeration constant (its position in its enum declaration, where the initial constant is assigned an ordinal of zero). Most programmers will have no use for this method. It is designed for use by sophisticated enum-based data structures, such as `EnumSet` and `EnumMap`.
- **Beginner-Friendly Explanation:** `ordinal()` tells you the number of the constant, starting at 0. The first constant has ordinal 0, the second has 1, and so on.

**Purposes**

- To obtain the position of a constant for use in array-based data structures.
- To support `EnumSet` and `EnumMap` internally.
- To provide a stable identifier for serialization purposes.

**Syntax Structures and Rules**

Complete general syntax:

```java
int position = EnumType.CONSTANT.ordinal();
```

Component breakdown:

- Returns an `int` representing the position.

Syntax rules: The ordinal is assigned in declaration order.

Constraints and limitations: The Java Language Specification §8.9 does not specify the use of `ordinal()` in programs. Relying on `ordinal()` is discouraged for general programming because it is fragile: if the declaration order changes, the ordinal values change. `ordinal()` should be used only with `EnumSet` and `EnumMap`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Using `ordinal()`**

```java
public enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE }

public class OrdinalExample {
    public static void main(String[] args) {
        for (Size s : Size.values()) {
            System.out.println(s + " has ordinal " + s.ordinal());
        }
    }
}
```

Expected Output:

```
SMALL has ordinal 0
MEDIUM has ordinal 1
LARGE has ordinal 2
EXTRA_LARGE has ordinal 3
```

**Real-World Cases with Explanation**

`ordinal()` is used internally by `EnumSet` and `EnumMap`. In application code, it should be avoided in favour of storing explicit IDs or codes in enum fields, as relying on ordinal values is fragile against reordering.

**References Links**

- Enum (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html
- Enum Constants (JLS §8.9.1) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9.1

#### 2.4 `name()` Method

**Definitions**

- **Core Definition:** Returns the exact name of the enum constant as declared in the enum declaration.
- **Technical Definition:** `public final String name()` returns the name of this enum constant, exactly as declared in its enum declaration. Most programmers should use the `toString()` method in preference to this one, as the `toString()` method may return a more user-friendly name. This method is designed primarily for use in specialized situations where correctness depends on getting the exact name, which will not vary from release to release.
- **Beginner-Friendly Explanation:** `name()` gives you the exact identifier of the constant, such as `"MONDAY"`. Unlike `toString()`, it cannot be overridden.

**Purposes**

- To obtain the exact declared name of an enum constant.
- To support serialization and reflection.
- To provide a stable identifier that does not change even if `toString()` is overridden.

**Syntax Structures and Rules**

Complete general syntax:

```java
String name = EnumType.CONSTANT.name();
```

Component breakdown:

- Returns the exact name as declared.

Syntax rules: The `name()` method is `final` and cannot be overridden.

Constraints and limitations: The name may not be user-friendly if `toString()` is overridden.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `name()` vs `toString()`**

```java
public enum Status {
    ACTIVE("Active Status"),
    INACTIVE("Inactive Status");

    private final String displayName;

    Status(String displayName) {
        this.displayName = displayName;
    }

    @Override
    public String toString() {
        return displayName;
    }

    public static void main(String[] args) {
        Status s = Status.ACTIVE;
        System.out.println("name(): " + s.name());
        System.out.println("toString(): " + s.toString());
    }
}
```

Expected Output:

```
name(): ACTIVE
toString(): Active Status
```

**Real-World Cases with Explanation**

`name()` is used in serialization frameworks where the exact constant name is required, in logging where a stable identifier is preferred, and in `Enum.valueOf()` which uses the exact name for lookup.

**References Links**

- Enum (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html

### 3. Advanced Features and Behavior

**Core Definition:** Advanced enum features include constant-specific class bodies (which allow each constant to have its own method implementation), interface implementation, use in switch statements and expressions, and inherent thread safety and serialization safety.

**Technical Definition:** The optional class body of an enum constant implicitly defines an anonymous class declaration that extends the immediately enclosing enum type. An enum class may implement interfaces. Enum types can be used in `switch` statements and, since Java 21, enum constants may be qualified in `switch` labels. Enum types are inherently thread-safe because their instances are initialized during class loading. Special treatment by the serialization mechanism ensures that duplicate instances are never created as a result of deserialization.

**Beginner-Friendly Explanation:** Enums can do more than just list constants. Each constant can have its own special behavior (constant-specific class bodies). Enums can implement interfaces so they can be used polymorphically. They work beautifully with `switch` statements. And they are safe to use in multi-threaded programs without extra synchronization.

#### 3.1 Constant-Specific Class Bodies

**Definitions**

- **Core Definition:** The ability to provide a separate class body for each enum constant, allowing each constant to override methods or provide its own implementation.
- **Technical Definition:** The optional class body of an enum constant implicitly defines an anonymous class declaration that extends the immediately enclosing enum type. When an enum constant has a class body, the enum is not implicitly `final`.
- **Beginner-Friendly Explanation:** You can give each constant its own version of a method. For example, a `PLUS` constant can add, while a `MINUS` constant can subtract, even though both are part of the same enum.

**Purposes**

- To provide distinct behavior for each enum constant.
- To implement the State pattern or Strategy pattern within an enum.
- To avoid long `if-else` or `switch` chains by encapsulating behavior in each constant.

**Syntax Structures and Rules**

Complete general syntax:

```java
public enum EnumName {
    CONSTANT1 {
        @Override
        public ReturnType method() {
            // implementation for CONSTANT1
        }
    },
    CONSTANT2 {
        @Override
        public ReturnType method() {
            // implementation for CONSTANT2
        }
    };

    public abstract ReturnType method();
}
```

Component breakdown:

- `CONSTANT1 { ... }`: the constant with its class body.
- `public abstract ReturnType method()`: the abstract method declared in the enum.
- Each constant must provide an implementation.

Syntax rules: Enum declarations may not use the `abstract` modifier unless they contain constant-specific class bodies for every enum constant, and any abstract methods declared in the class body are overridden in all the constant-specific class bodies.

Constraints and limitations: The class body of an enum constant cannot contain constructors. It may contain fields and methods.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Constant-specific behavior**

```java
public enum Operation {
    PLUS {
        @Override
        public int apply(int a, int b) { return a + b; }
    },
    MINUS {
        @Override
        public int apply(int a, int b) { return a - b; }
    },
    TIMES {
        @Override
        public int apply(int a, int b) { return a * b; }
    };

    public abstract int apply(int a, int b);

    public static void main(String[] args) {
        System.out.println("PLUS: " + Operation.PLUS.apply(3, 4));
        System.out.println("MINUS: " + Operation.MINUS.apply(10, 4));
        System.out.println("TIMES: " + Operation.TIMES.apply(3, 4));
    }
}
```

Expected Output:

```
PLUS: 7
MINUS: 6
TIMES: 12
```

**Why the code produces this result:** Each enum constant overrides the abstract `apply` method with its own implementation. When `apply` is called on a constant, the constant's specific implementation is invoked.

**Real-World Cases with Explanation**

Constant-specific class bodies are used to implement finite state machines, where each state has its own `handle` method; in arithmetic operation enums; and in tax calculation systems where each tax category has its own calculation logic.

**References Links**

- Enum Body Declarations (JLS §8.9.2) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9.2
- The State Pattern (Design Patterns Reloaded) – https://github.com/forax/design-pattern-reloaded

#### 3.2 Interface Implementation by Enums

**Definitions**

- **Core Definition:** The ability of an enum class to implement one or more interfaces, allowing enum constants to be used polymorphically.
- **Technical Definition:** An enum class may declare superinterfaces. Each enum constant inherits the interface implementations from the enum class, or can override them via constant-specific class bodies.
- **Beginner-Friendly Explanation:** An enum can implement an interface, which means you can use enum constants wherever the interface type is expected. This makes enums very flexible.

**Purposes**

- To allow enum constants to be used polymorphically through interface types.
- To enforce a common contract across enum constants.
- To enable enums to participate in framework-based designs.

**Syntax Structures and Rules**

Complete general syntax:

```java
public interface SomeInterface {
    ReturnType method();
}

public enum EnumName implements SomeInterface {
    CONSTANT1, CONSTANT2;

    @Override
    public ReturnType method() {
        // shared implementation
    }
}
```

Component breakdown:

- `implements SomeInterface`: declares that the enum implements the interface.
- Each constant inherits the interface method.

Syntax rules: An enum can implement multiple interfaces. The interface methods must be implemented either in the enum class body or in each constant's class body.

Constraints and limitations: An enum cannot extend a class, but it can implement interfaces.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Enum implementing an interface**

```java
public interface Describable {
    String describe();
}

public enum Shape implements Describable {
    CIRCLE {
        @Override
        public String describe() { return "A round shape"; }
    },
    SQUARE {
        @Override
        public String describe() { return "A shape with four equal sides"; }
    };

    public static void main(String[] args) {
        for (Shape s : values()) {
            System.out.println(s + ": " + s.describe());
        }
    }
}
```

Expected Output:

```
CIRCLE: A round shape
SQUARE: A shape with four equal sides
```

**Real-World Cases with Explanation**

Enums implementing interfaces are used in plugin architectures where each enum constant represents a plugin that conforms to a common interface, and in command patterns where each enum constant represents a command with an `execute()` method.

**References Links**

- Enum Body Declarations (JLS §8.9.2) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9.2
- Why would an Enum implement an Interface? – https://stackoverflow.com/questions/2709593/why-would-an-enum-implement-an-interface

#### 3.3 Enums in Switch Statements and Expressions

**Definitions**

- **Core Definition:** The use of enum types as the selector expression in `switch` statements and expressions.
- **Technical Definition:** An enum type can be used in a `switch` statement. Since Java 21, enum constants may be qualified in `switch` labels. The `case` labels must be unqualified enum constant names when the switch selector is an enum type.
- **Beginner-Friendly Explanation:** You can use an enum in a `switch` to run different code for different constants. This is one of the most common uses of enums.

**Purposes**

- To execute different code paths based on the enum constant.
- To provide a readable alternative to long `if-else` chains.
- To leverage exhaustiveness checking in switch expressions.

**Syntax Structures and Rules**

Complete general syntax:

```java
switch (enumVariable) {
    case CONSTANT1 -> // action
    case CONSTANT2 -> // action
    default -> // action
}
```

Component breakdown:

- `enumVariable`: the enum-typed variable.
- `case CONSTANT1`: the unqualified constant name.

Syntax rules: The switch selector must be of an enum type. The `case` labels must be unqualified enum constant names (e.g., `case MONDAY:`, not `case Day.MONDAY:`). Since Java 21, fully-qualified enum names in switch labels compile and work fine.

Constraints and limitations: A `default` clause is not required if all constants are covered (in switch expressions). In switch statements, a `default` clause is optional.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Enum in switch statement**

```java
public enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

public class SwitchEnumExample {
    public static void main(String[] args) {
        Day day = Day.WEDNESDAY;
        switch (day) {
            case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY ->
                System.out.println("Weekday");
            case SATURDAY, SUNDAY ->
                System.out.println("Weekend");
        }
    }
}
```

Expected Output:

```
Weekday
```

**Example 2: Enum in switch expression**

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        return switch (this) {
            case PLUS -> x + y;
            case MINUS -> x - y;
            case TIMES -> x * y;
            case DIVIDE -> x / y;
        };
    }

    public static void main(String[] args) {
        System.out.println(Operation.PLUS.apply(10, 5));
    }
}
```

Expected Output:

```
15.0
```

**Real-World Cases with Explanation**

Enums in switch statements are used in command processors, state machines, and any scenario where behavior depends on a finite set of values. Switch expressions with enums are exhaustive, meaning the compiler verifies that all constants are handled.

**References Links**

- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html
- Primitive Types in Patterns, instanceof, and switch (Java SE 21) – https://docs.oracle.com/en/java/javase/21/language/pattern-matching-switch.html

#### 3.4 Inherent Thread Safety and Serialization Safety

**Definitions**

- **Core Definition:** Enum types are inherently thread-safe because their instances are created during class initialization, which is thread-safe; and they have special serialization handling that prevents duplicate instances.
- **Technical Definition:** Enum constants are `static final` fields initialized in a static initializer block. Class loading and initialization are thread-safe. Additionally, the serialization mechanism treats enum constants specially: the `final` `clone` method in `Enum` ensures that enum constants can never be cloned, reflective instantiation of enum classes is prohibited, and special treatment by the serialization mechanism ensures that duplicate instances are never created as a result of deserialization.
- **Beginner-Friendly Explanation:** You do not need to worry about multiple threads creating duplicate enum constants. Java handles this automatically. And if you serialize an enum and then deserialize it, you get the exact same instance back, not a copy.

**Purposes**

- To provide a thread-safe singleton without explicit synchronization.
- To ensure that serialization and deserialization preserve the singleton property.
- To eliminate the risk of duplicate instances in concurrent environments.

**Syntax Structures and Rules**

Complete general syntax:

```java
public enum EnumName {
    CONSTANT1, CONSTANT2;
    // No special syntax required; thread safety is inherent.
}
```

Component breakdown:

- Enum constants are initialized in a static block, which is thread-safe.

Syntax rules: No explicit synchronization is needed for enum constant initialization. However, methods added to an enum class do not carry any thread safety guarantee unless declared explicitly.

Constraints and limitations: While enum instance creation is thread-safe, mutable state within enum constants requires external synchronization if accessed concurrently.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Enum singleton thread safety**

```java
public enum Singleton {
    INSTANCE;

    private int counter = 0;

    public synchronized void increment() {
        counter++;
    }

    public int getCounter() { return counter; }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) Singleton.INSTANCE.increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) Singleton.INSTANCE.increment();
        });
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println("Counter: " + Singleton.INSTANCE.getCounter());
    }
}
```

Expected Output:

```
Counter: 2000
```

**Why the code produces this result:** The `INSTANCE` is a single shared instance. The `increment` method is `synchronized` to protect the mutable `counter` field. The enum itself guarantees that only one `INSTANCE` exists.

**Real-World Cases with Explanation**

Enum singletons are used for configuration managers, connection pools, and caches where a single instance must be shared across the application and serialization safety is important.

**References Links**

- Enum (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html
- SEI CERT Oracle Coding Standard for Java – https://wiki.sei.cmu.edu/confluence/display/java/OBJ03-J.+Prevent+the+creation+of+extraneous+objects

### 4. Specialized Collections

**Core Definition:** `EnumSet` and `EnumMap` are specialized collection implementations designed specifically for enum types, offering high performance and compact memory usage.

**Technical Definition:** `EnumSet<E extends Enum<E>>` is a specialized `Set` implementation for use with enum types. All of the elements in an enum set must come from a single enum type that is specified, explicitly or implicitly, when the set is created. Enum sets are represented internally as bit vectors. `EnumMap<K extends Enum<K>, V>` is a specialized `Map` implementation for use with enum type keys. Enum maps are represented internally as arrays. All basic operations execute in constant time.

**Beginner-Friendly Explanation:** `EnumSet` is a very fast set for enums. `EnumMap` is a very fast map where the keys are enums. Both are more efficient than regular `HashSet` and `HashMap` when working with enums.

#### 4.1 `EnumSet`

**Definitions**

- **Core Definition:** A specialized `Set` implementation for enum types, internally represented as a bit vector.
- **Technical Definition:** `public abstract class EnumSet<E extends Enum<E>> extends AbstractSet<E> implements Cloneable, Serializable`. All of the elements in an enum set must come from a single enum type. Enum sets are represented internally as bit vectors. This representation is extremely compact and efficient. Null elements are not permitted. Attempts to insert a `null` element will throw `NullPointerException`. `EnumSet` is not synchronized.
- **Beginner-Friendly Explanation:** `EnumSet` is a set that can only hold values from one enum. It is extremely fast and memory-efficient because it uses bits to represent which constants are present.

**Purposes**

- To provide a high-performance set implementation for enum types.
- To replace traditional `int`-based "bit flags" with a typesafe alternative.
- To support efficient bulk operations such as `removeAll` and `retainAll` using bitwise arithmetic.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
EnumSet<EnumType> set = EnumSet.noneOf(EnumType.class);
EnumSet<EnumType> set = EnumSet.allOf(EnumType.class);
EnumSet<EnumType> set = EnumSet.of(EnumType.CONSTANT1, EnumType.CONSTANT2);
EnumSet<EnumType> set = EnumSet.range(EnumType.CONSTANT1, EnumType.CONSTANT3);
EnumSet<EnumType> set = EnumSet.complementOf(otherSet);
```

Component breakdown:

- `noneOf`: creates an empty enum set.
- `allOf`: creates a set containing all constants.
- `of`: creates a set containing the specified constants.
- `range`: creates a set containing all constants in the specified range.
- `complementOf`: creates a set containing all constants not in the specified set.

Syntax rules: All elements must come from a single enum type. Null elements are not permitted. `EnumSet` is not synchronized.

Constraints and limitations: `EnumSet` is not synchronized. If multiple threads access it concurrently, external synchronization is required.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `EnumSet` operations**

```java
import java.util.EnumSet;

public enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

public class EnumSetExample {
    public static void main(String[] args) {
        EnumSet<Day> weekdays = EnumSet.range(Day.MONDAY, Day.FRIDAY);
        System.out.println("Weekdays: " + weekdays);

        EnumSet<Day> weekend = EnumSet.complementOf(weekdays);
        System.out.println("Weekend: " + weekend);

        EnumSet<Day> all = EnumSet.allOf(Day.class);
        System.out.println("All days: " + all);
    }
}
```

Expected Output:

```
Weekdays: [MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY]
Weekend: [SATURDAY, SUNDAY]
All days: [MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY]
```

**Real-World Cases with Explanation**

`EnumSet` is used to represent sets of permissions (e.g., `READ`, `WRITE`, `EXECUTE`), days of the week for scheduling, and feature flags in configuration systems.

**References Links**

- EnumSet (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumSet.html
- EnumSet and EnumMap (TechEmpower) – https://www.techempower.com/blog/2017/02/14/enumset-and-enummap/

#### 4.2 `EnumMap`

**Definitions**

- **Core Definition:** A specialized `Map` implementation for enum type keys, internally represented as an array.
- **Technical Definition:** `public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V> implements Serializable, Cloneable`. All of the keys in an enum map must come from a single enum type that is specified, explicitly or implicitly, when the map is created. Enum maps are represented internally as arrays. This representation is extremely compact and efficient. Enum maps are maintained in the natural order of their keys (the order in which the enum constants are declared). Null keys are not permitted. Null values are permitted. `EnumMap` is not synchronized.
- **Beginner-Friendly Explanation:** `EnumMap` is a map where the keys are enum constants. It is faster and more memory-efficient than a regular `HashMap` when the keys are enums.

**Purposes**

- To provide a high-performance map implementation for enum keys.
- To maintain keys in their natural declaration order.
- To support constant-time basic operations.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
EnumMap<EnumType, ValueType> map = new EnumMap<>(EnumType.class);
EnumMap<EnumType, ValueType> map = new EnumMap<>(otherMap);
```

Component breakdown:

- `EnumType.class`: the class object of the enum type used as keys.
- The map is created with the specified key type.

Syntax rules: Null keys are not permitted and throw `NullPointerException`. Null values are permitted. Iterators are weakly consistent.

Constraints and limitations: `EnumMap` is not synchronized. If multiple threads access it concurrently, external synchronization is required.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `EnumMap` operations**

```java
import java.util.EnumMap;

public enum Season { WINTER, SPRING, SUMMER, FALL }

public class EnumMapExample {
    public static void main(String[] args) {
        EnumMap<Season, String> map = new EnumMap<>(Season.class);
        map.put(Season.WINTER, "Cold");
        map.put(Season.SPRING, "Warm");
        map.put(Season.SUMMER, "Hot");
        map.put(Season.FALL, "Cool");

        for (Season s : map.keySet()) {
            System.out.println(s + " -> " + map.get(s));
        }
    }
}
```

Expected Output:

```
WINTER -> Cold
SPRING -> Warm
SUMMER -> Hot
FALL -> Cool
```

**Why the code produces this result:** `EnumMap` maintains keys in their natural declaration order. The `keySet()` iterator returns keys in that order.

**Real-World Cases with Explanation**

`EnumMap` is used in dispatch tables where each enum constant maps to a handler, in configuration systems where enum keys map to settings, and in game development where enum types map to properties or behaviors.

**References Links**

- EnumMap (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumMap.html
- EnumSet and EnumMap (TechEmpower) – https://www.techempower.com/blog/2017/02/14/enumset-and-enummap/

### 5. Design Patterns and Use Cases

**Core Definition:** Java enums are particularly well-suited for implementing certain design patterns, most notably the Singleton pattern and the State pattern, due to their inherent guarantees of single instances and their support for polymorphic behavior.

**Technical Definition:** The Singleton pattern ensures that a class has only one instance and provides a global point of access to it. The State pattern allows an object to alter its behavior when its internal state changes. Enum types are especially suited to these patterns because they provide an ironclad guarantee against multiple instantiation (for the Singleton pattern) and constant-specific behavior (for the State pattern).

**Beginner-Friendly Explanation:** Enums are naturally good at being singletons (there is only one of each constant) and at being states (each constant can have its own behavior).

#### 5.1 Singleton Pattern Implementation via Enum

**Definitions**

- **Core Definition:** Using a single-element enum to implement the Singleton pattern.
- **Technical Definition:** A single-element enum type is the best way to implement a singleton. This approach is functionally equivalent to the public field approach, except that it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks.
- **Beginner-Friendly Explanation:** You can create a singleton by writing an enum with one constant, like `INSTANCE`. Java guarantees that there is only one instance, and it works with serialization and threads automatically.

**Purposes**

- To provide a thread-safe singleton without explicit synchronization.
- To guarantee a single instance even in the presence of serialization and reflection.
- To eliminate the need for private constructors, static fields, and synchronized methods.

**Syntax Structures and Rules**

Complete general syntax:

```java
public enum Singleton {
    INSTANCE;

    // methods and fields
}
```

Component breakdown:

- `INSTANCE`: the single enum constant.
- Methods and fields are declared as usual.

Syntax rules: No additional synchronization or serialization handling is needed. The singleton is accessed via `Singleton.INSTANCE`.

Constraints and limitations: The singleton is initialized when the enum class is loaded, which is typically when the first time it is accessed. If lazy initialization is required, an enum singleton is not suitable.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Enum singleton**

```java
public enum Singleton {
    INSTANCE;

    private int value;

    public void setValue(int value) { this.value = value; }
    public int getValue() { return value; }

    public static void main(String[] args) {
        Singleton s1 = Singleton.INSTANCE;
        Singleton s2 = Singleton.INSTANCE;

        s1.setValue(42);
        System.out.println("s1 value: " + s1.getValue());
        System.out.println("s2 value: " + s2.getValue());
        System.out.println("Same instance: " + (s1 == s2));
    }
}
```

Expected Output:

```
s1 value: 42
s2 value: 42
Same instance: true
```

**Why the code produces this result:** Both `s1` and `s2` refer to the same `INSTANCE`. Setting a value through `s1` is visible through `s2` because they are the same object.

**Real-World Cases with Explanation**

Enum singletons are used for configuration managers, connection pools, thread pools, and caches. They are recommended by Joshua Bloch in *Effective Java* as "the best way to implement a singleton."

**References Links**

- SEI CERT Oracle Coding Standard for Java – https://wiki.sei.cmu.edu/confluence/display/java/OBJ03-J.+Prevent+the+creation+of+extraneous+objects
- Effective Java (Joshua Bloch) – https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/

#### 5.2 State Pattern Implementation via Enum

**Definitions**

- **Core Definition:** Using an enum to represent the different states of a state machine, with each state encapsulated in an enum constant.
- **Technical Definition:** The State pattern allows an object to alter its behavior when its internal state changes. An enum can model states where each constant represents a distinct state and carries its own behavior via constant-specific class bodies.
- **Beginner-Friendly Explanation:** An enum can represent the different states of a system, like the states of an order (CREATED, PAID, SHIPPED). Each state can have its own rules for what transitions are allowed.

**Purposes**

- To model finite state machines in a type-safe way.
- To encapsulate state-specific behavior within each enum constant.
- To eliminate complex conditional logic for state transitions.

**Syntax Structures and Rules**

Complete general syntax:

```java
public enum State {
    STATE_A {
        @Override
        public State next() { return STATE_B; }
    },
    STATE_B {
        @Override
        public State next() { return STATE_C; }
    },
    STATE_C {
        @Override
        public State next() { return STATE_A; }
    };

    public abstract State next();
}
```

Component breakdown:

- Each enum constant represents a state.
- Each constant can override methods to define state-specific behavior.

Syntax rules: The enum can have fields to hold state data. Transitions can be modeled as methods.

Constraints and limitations: The enum-based State pattern is best suited for simple state machines. Complex state machines with many states and transitions may benefit from a class-based approach.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Simple state machine**

```java
public enum TrafficLight {
    RED {
        @Override
        public TrafficLight next() { return GREEN; }
    },
    GREEN {
        @Override
        public TrafficLight next() { return YELLOW; }
    },
    YELLOW {
        @Override
        public TrafficLight next() { return RED; }
    };

    public abstract TrafficLight next();

    public static void main(String[] args) {
        TrafficLight light = RED;

        for (int i = 0; i < 5; i++) {
            System.out.println(light);
            light = light.next();
        }
    }
}
```

Expected Output:

```
RED
GREEN
YELLOW
RED
GREEN
```

**Real-World Cases with Explanation**

Enum-based state machines are used in order processing systems, game character states, network protocol states, and workflow engines.

**References Links**

- The State Pattern (Design Patterns Reloaded) – https://github.com/forax/design-pattern-reloaded
- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html

## Summary Table of Key Features

| Feature | Description | Key Benefit |
|---|---|---|
| `enum` keyword | Declares a fixed set of constants | Type safety |
| Enum constants | Named instances of the enum class | Singleton per constant |
| Instance fields | Data associated with each constant | Custom metadata |
| Private constructors | Initialize constant fields | Controlled setup |
| Custom methods | Behavior shared by all constants | Encapsulation |
| `values()` | Returns all constants | Iteration support |
| `valueOf()` | Looks up constant by name | String conversion |
| `ordinal()` | Returns position (0-based) | EnumSet/EnumMap support |
| `name()` | Returns exact constant name | Stable identifier |
| Constant-specific bodies | Per-constant method implementations | Polymorphic behavior |
| Interface implementation | Enums can implement interfaces | Framework integration |
| Switch support | Enums work in switch statements/expressions | Readable branching |
| Thread safety | Inherent thread safety | No synchronization needed |
| Serialization safety | Special serialization handling | No duplicate instances |
| `EnumSet` | Bit-vector set for enums | Extremely fast |
| `EnumMap` | Array-based map for enum keys | Constant-time operations |
| Singleton pattern | Single-element enum | Best-practice singleton |
| State pattern | Enum-based state machine | Clean state modeling |

## Deprecated, Unsafe, and Version-Specific Notes

- The `int Enum` pattern (using `public static final int` constants) is deprecated in favour of enum types. It lacks type safety, namespace, and compile-time safety.
- `ordinal()` is designed for use by sophisticated enum-based data structures such as `EnumSet` and `EnumMap`. The Java Language Specification §8.9 does not specify the use of `ordinal()` in programs, and relying on it is fragile because it depends on declaration order.
- Enum types cannot be extended. An enum declaration is implicitly `final` unless it contains at least one enum constant with a class body.
- Enum constructors are implicitly `private` and cannot be `public` or `protected`. It is a compile-time error to use any access modifier other than `private` on an enum constructor.
- Since Java 21, enum constants may be qualified in `switch` labels. Prior to Java 21, only unqualified enum constant names were permitted in `case` labels.
- `EnumSet` and `EnumMap` are not synchronized. If multiple threads access them concurrently, external synchronization is required.
- Enum methods added to an enum class do not carry any thread safety guarantee unless declared explicitly. Only the creation of enum constants is thread-safe.

## References

- Enum Types (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html
- Enum Classes (Java Language Specification, Java SE 21, §8.9) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9
- Enum (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html
- EnumSet (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumSet.html
- EnumMap (Java SE 21 API Documentation) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumMap.html
- Enum Types (Oracle Technology Network) – https://docs.oracle.com/javase/jp/8/docs/technotes/guides/language/enums.html
- Specification for JEP 301: Enhanced Enums – https://cr.openjdk.org/~dlsmith/jep301/enhanced-enums.html
- SEI CERT Oracle Coding Standard for Java – https://wiki.sei.cmu.edu/confluence/display/java/OBJ03-J.+Prevent+the+creation+of+extraneous+objects
- Effective Java (Joshua Bloch, 3rd Edition) – https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/
- EnumSet and EnumMap (TechEmpower) – https://www.techempower.com/blog/2017/02/14/enumset-and-enummap/
- The State Pattern (Design Patterns Reloaded) – https://github.com/forax/design-pattern-reloaded