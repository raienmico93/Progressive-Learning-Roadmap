# Java Variables: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A variable is a named container that holds a value of a specific data type during program execution. In Java, variables are strongly typed, meaning every variable must be declared with a type before use.

**Technical Definition:** "A variable is a container that holds a value while a Java program is running. A variable is assigned a data type that designates the type and quantity of value it can hold". "The Java programming language defines the following kinds of variables: Instance Variables (Non-Static Fields), Class Variables (Static Fields), Local Variables, and Parameters". "Every variable in Java has a data type. The data type determines the values that the variable can hold, the operations that can be performed on the variable, and the amount of memory allocated".

**Beginner-Friendly Explanation:** Think of a variable as a labelled box where you can store things. The label is the variable name, and the type of box determines what you can put inside — a box labelled "int" can only hold whole numbers, a box labelled "String" can only hold text. You can put something in the box (assignment), look inside (access), or replace what's in the box (reassignment). Java is strict: once you declare a box as "int", you can't put a "String" in it.

### Key Characteristics

- **Strongly Typed:** Every variable must have a declared type.
- **Named:** Variables are identified by valid Java identifiers.
- **Scoped:** Variables have a limited region of visibility.
- **Typed:** The type determines the size, range, and operations.
- **Stored:** Variables are stored on the stack (local) or heap (instance/static).
- **Initialised:** Variables must be initialised before use (local variables explicitly; others get defaults).
- **Immutable Option:** `final` variables cannot be reassigned after initialisation.
- **Type-Inferred Option:** `var` (Java 10+) infers the type from the initialiser.

### Prerequisites

- Basic understanding of Java syntax
- Familiarity with data types (primitives and references)
- Knowledge of classes and methods
- Understanding of scope and lifetime concepts

### Related Programming Areas

- **Data Types:** Primitive types (`int`, `double`, `boolean`, etc.) and reference types.
- **Methods:** Parameters are a special kind of local variable.
- **Classes and Objects:** Instance and static variables are members of classes.
- **Memory Management:** Stack vs heap allocation.
- **Immutability:** `final` variables and constants.

### Core Concepts / Features

---

## 1. Variables

### Definitions

**Core Definition:** A variable is a named storage location in memory that holds a value of a specific type.

**Technical Definition:** "A variable is a container that holds a value while a Java program is running. A variable is assigned a data type that designates the type and quantity of value it can hold". "The Java programming language is statically-typed, which means that every variable must have a data type before it can be used".

**Beginner-Friendly Explanation:** A variable is like a labelled jar. The label is the name, the jar's shape is the type, and the contents are the value. You can label a jar "cookies" (int) and put cookies in it, but you can't put soup (String) in a cookie jar.

### Purposes

- To store data values for use during program execution
- To give meaningful names to values for readability
- To allow values to be changed (mutability) or fixed (immutability)
- To manage memory allocation for data
- To enable data sharing between methods and objects

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Declaration
type variableName;

// Initialization
variableName = value;

// Declaration + Initialization
type variableName = value;

// Multiple declarations
type var1, var2, var3;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `type` | The data type (e.g., `int`, `String`, `var`). |
| `variableName` | A valid Java identifier. |
| `value` | An expression compatible with the type. |

**Syntax Rules:**

- Variable names must be valid Java identifiers (letters, digits, `_`, `$`, not starting with a digit).
- Variable names are case-sensitive (`count` and `Count` are different).
- Variables must be declared before use.
- Local variables must be explicitly initialised before use.
- Instance and static variables get default values if not initialised.

**Constraints and Limitations:**

- Reserved keywords (e.g., `int`, `class`, `if`) cannot be used as variable names.
- Variable names cannot contain spaces or special characters (except `_` and `$`).
- The type of a variable cannot be changed after declaration.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Variable Declaration and Initialization**

```java
// File: VariableBasicDemo.java
public class VariableBasicDemo {
    public static void main(String[] args) {
        // Declaration
        int age;

        // Initialization
        age = 25;

        // Declaration + Initialization
        String name = "Alice";

        // Multiple variables of the same type
        int x = 10, y = 20, z = 30;

        // Using the variables
        System.out.println("Name : " + name);    // Name: Alice
        System.out.println("Age  : " + age);      // Age: 25
        System.out.println("Sum  : " + (x + y + z));  // Sum: 60
    }
}
```

**Expected Output:**
```
Name : Alice
Age  : 25
Sum  : 60
```

**Step-by-Step Explanation:**

1. `int age;` declares a variable of type `int` without initialising it.
2. `age = 25;` assigns the value `25` to `age`.
3. `String name = "Alice";` declares and initialises a `String` variable.
4. `int x = 10, y = 20, z = 30;` declares and initialises three `int` variables.
5. The variables are used in `System.out.println()` statements.

### Real-World Cases

- **Configuration Values:** Storing database URLs, port numbers, and API keys.
- **User Input:** Storing values entered by users.
- **Computed Results:** Storing the output of calculations.
- **State Tracking:** Storing the current state of an object or application.

---

## 2. Variable Declaration

### Definitions

**Core Definition:** Variable declaration is the statement that introduces a variable by specifying its type and name.

**Technical Definition:** "A variable declaration specifies the type of the variable and its name". The declaration tells the compiler to reserve memory for the variable and associate it with the given name and type.

**Beginner-Friendly Explanation:** Declaring a variable is like buying a new labelled box. You tell the compiler "I need a box for whole numbers, and I'll call it `count`." From that point on, the compiler knows about `count` and what kind of data it can hold.

### Purposes

- To introduce a new variable into the program
- To allocate memory for the variable
- To associate a name with a type
- To enable type checking by the compiler

### Syntax Rules and Structure

```java
type variableName;
type variableName1, variableName2, ...;
```

**Syntax Rules:**

- The type must be a valid Java type (primitive or reference).
- The variable name must be a valid identifier.
- Multiple variables of the same type can be declared in one statement, separated by commas.
- Declaration without initialisation is allowed for instance and static variables (they get default values) but not for local variables.

### Annotated Code Example

```java
// File: DeclarationDemo.java
public class DeclarationDemo {
    // Instance variables: declared without initialization
    private int instanceCount;
    private String instanceName;

    // Static variable: declared without initialization
    private static double staticValue;

    public static void main(String[] args) {
        // Local variable: must be initialized before use
        int localVar = 10;

        // Multiple declarations
        int a, b, c;
        a = 1; b = 2; c = 3;

        System.out.println("Local: " + localVar);
        System.out.println("Instance count: " + new DeclarationDemo().instanceCount);  // 0
        System.out.println("Static value: " + staticValue);  // 0.0
    }
}
```

**Expected Output:**
```
Local: 10
Instance count: 0
Static value: 0.0
```

### Real-World Cases

- **Class Fields:** Declaring instance variables in a class.
- **Method Locals:** Declaring temporary variables in methods.
- **Configuration:** Declaring static constants.

---

## 3. Variable Initialization

### Definitions

**Core Definition:** Variable initialization is the process of assigning an initial value to a variable.

**Technical Definition:** "Initialization is the process of assigning a value to a variable for the first time". Local variables must be explicitly initialised before use; instance and static variables are automatically initialised to default values.

**Beginner-Friendly Explanation:** Initialization is putting something in the box for the first time. You can either put something in when you buy the box (declaration + initialization) or buy an empty box and fill it later (declaration then initialization).

### Purposes

- To give a variable its starting value
- To ensure the variable has a known value before use
- To avoid `NullPointerException` and undefined behaviour
- To set up the initial state of an object

### Syntax Rules and Structure

```java
// Declaration + initialization
int x = 10;

// Separate initialization
int y;
y = 20;

// Initialization with an expression
int z = x + y;

// Multiple initialization
int a = 1, b = 2, c = 3;
```

**Default Values Table:**

| Type | Default Value |
|------|---------------|
| `byte`, `short`, `int`, `long` | `0` |
| `float`, `double` | `0.0` |
| `char` | `'\u0000'` |
| `boolean` | `false` |
| Reference types | `null` |

**Syntax Rules:**

- Local variables must be initialised before use; otherwise, a compile-time error occurs.
- Instance and static variables are automatically initialised to default values.
- Initialisation can happen at declaration or in a constructor/initialiser block.

### Annotated Code Example

```java
// File: InitializationDemo.java
public class InitializationDemo {
    // Instance variable: initialized at declaration
    private int count = 10;

    // Instance variable: initialized in constructor
    private String name;

    // Static variable: initialized at declaration
    private static final double PI = 3.14159;

    public InitializationDemo(String name) {
        this.name = name;  // Initialized in constructor
    }

    public static void main(String[] args) {
        InitializationDemo demo = new InitializationDemo("Test");
        System.out.println("Count: " + demo.count);   // 10
        System.out.println("Name: " + demo.name);     // Test
        System.out.println("PI: " + PI);               // 3.14159

        // Local variable: must be initialized
        int local;
        // System.out.println(local);  // Compile error!
        local = 42;
        System.out.println("Local: " + local);         // 42
    }
}
```

**Expected Output:**
```
Count: 10
Name: Test
PI: 3.14159
Local: 42
```

### Real-World Cases

- **Object Construction:** Initialising fields in constructors.
- **Configuration:** Initialising constants and configuration values.
- **Computation:** Initialising accumulators and counters.

---

## 4. Variable Assignment

### Definitions

**Core Definition:** Variable assignment is the operation of changing the value stored in a variable.

**Technical Definition:** "Assignment is the process of storing a value in a variable". The assignment operator `=` evaluates the right-hand expression and stores the result in the left-hand variable.

**Beginner-Friendly Explanation:** Assignment is replacing what's in the box with something new. You can do this as many times as you want (unless the variable is `final`). The old value is discarded, and the new value takes its place.

### Purposes

- To change the value of a variable during program execution
- To update counters, accumulators, and state variables
- To store computed results
- To swap values between variables

### Syntax Rules and Structure

```java
variableName = expression;
```

**Compound Assignment Operators:**

| Operator | Equivalent To |
|----------|---------------|
| `+=` | `x = x + y` |
| `-=` | `x = x - y` |
| `*=` | `x = x * y` |
| `/=` | `x = x / y` |
| `%=` | `x = x % y` |
| `&=` | `x = x & y` |
| `\|=` | `x = x \| y` |
| `^=` | `x = x ^ y` |
| `<<=` | `x = x << y` |
| `>>=` | `x = x >> y` |

**Syntax Rules:**

- The left-hand side must be a variable (not an expression).
- The right-hand side must be compatible with the variable's type.
- Compound assignment operators combine an operation with assignment.
- The assignment expression itself has a value (the assigned value).

### Annotated Code Example

```java
// File: AssignmentDemo.java
public class AssignmentDemo {
    public static void main(String[] args) {
        int x = 10;

        // Simple assignment
        x = 20;
        System.out.println("x = " + x);  // 20

        // Compound assignment
        x += 5;
        System.out.println("x += 5: " + x);  // 25

        x -= 3;
        System.out.println("x -= 3: " + x);  // 22

        x *= 2;
        System.out.println("x *= 2: " + x);  // 44

        x /= 4;
        System.out.println("x /= 4: " + x);  // 11

        // Chained assignment
        int a, b, c;
        a = b = c = 100;
        System.out.println("a=" + a + ", b=" + b + ", c=" + c);  // 100, 100, 100
    }
}
```

**Expected Output:**
```
x = 20
x += 5: 25
x -= 3: 22
x *= 2: 44
x /= 4: 11
a=100, b=100, c=100
```

### Real-World Cases

- **Counters:** `count++` or `count += 1`.
- **Accumulators:** `total += price`.
- **State Updates:** Changing object state.
- **Swap Operations:** Exchanging values between variables.

---

## 5. Local Variables

### Definitions

**Core Definition:** Local variables are variables declared inside a method, constructor, or block, accessible only within that scope.

**Technical Definition:** "Local Variables: Similar to how an object stores its state in fields, a method will often store its temporary state in local variables. The syntax for declaring a local variable is similar to declaring a field (for example, `int count = 0;`). There is no special keyword designating a variable as local; that determination comes entirely from the location in which the variable is declared — which is between the opening and closing braces of a method. As such, local variables are only visible to the methods in which they are declared; they are not accessible from the rest of the class".

**Beginner-Friendly Explanation:** Local variables are like scratch paper used inside a single method. They're created when the method starts, used during the method, and thrown away when the method ends. No other method can see them.

### Purposes

- To store temporary values during method execution
- To hold intermediate results of computations
- To store method parameters
- To keep method state private to the method

### Syntax Rules and Structure

```java
void myMethod() {
    int localVar = 10;  // Local variable
    // Only accessible within myMethod
}
```

**Syntax Rules:**

- Local variables are declared inside a method, constructor, or block.
- They must be initialised before use.
- They are not accessible outside their scope.
- They cannot use access modifiers (`public`, `private`, etc.).
- They are stored on the stack.

**Constraints and Limitations:**

- Local variables are not automatically initialised.
- They cannot be `static`.
- They are destroyed when the method/block exits.

### Annotated Code Example

```java
// File: LocalVariableDemo.java
public class LocalVariableDemo {
    public static void main(String[] args) {
        // Local variable in main method
        int mainVar = 10;
        System.out.println("Main variable: " + mainVar);

        // Local variable in a block
        {
            int blockVar = 20;
            System.out.println("Block variable: " + blockVar);
        }
        // System.out.println(blockVar);  // Compile error: out of scope

        // Local variable in a loop
        for (int i = 0; i < 3; i++) {
            int loopVar = i * 10;
            System.out.println("Loop variable: " + loopVar);
        }
        // System.out.println(i);  // Compile error: out of scope
    }

    static void anotherMethod() {
        // mainVar is not accessible here
        int anotherVar = 30;
        System.out.println("Another method: " + anotherVar);
    }
}
```

**Expected Output:**
```
Main variable: 10
Block variable: 20
Loop variable: 0
Loop variable: 10
Loop variable: 20
```

### Real-World Cases

- **Method Parameters:** Variables passed to methods.
- **Loop Counters:** `for (int i = 0; ...)`.
- **Temporary Storage:** Holding values during calculations.
- **Exception Handling:** Variables in `try`/`catch` blocks.

---

## 6. Instance Variables

### Definitions

**Core Definition:** Instance variables (non-static fields) are variables declared in a class but outside any method, constructor, or block, with each instance of the class having its own copy.

**Technical Definition:** "Instance Variables (Non-Static Fields): Technically speaking, objects store their individual states in 'non-static fields', that is, fields declared without the `static` keyword. Non-static fields are also known as instance variables because their values are unique to each instance of a class". "Instance variables are initialized when the class is instantiated. They are accessible from any method, constructor, or block in the class".

**Beginner-Friendly Explanation:** Instance variables are like personal belongings of each object. Each `Person` object has its own `name` and `age` — changing one person's name doesn't affect another's.

### Purposes

- To store the state of an object
- To give each object its own copy of data
- To define the properties of a class
- To enable object-specific behaviour

### Syntax Rules and Structure

```java
public class MyClass {
    // Instance variables
    private int count;
    private String name;
    protected double value;
    public boolean flag;
}
```

**Syntax Rules:**

- Declared in a class but outside methods, constructors, and blocks.
- Can have access modifiers (`public`, `protected`, `private`, or default).
- Can be `final`, `volatile`, or `transient`.
- Are automatically initialised to default values.
- Are created when an object is instantiated.
- Are destroyed when the object is garbage collected.

### Annotated Code Example

```java
// File: InstanceVariableDemo.java
public class InstanceVariableDemo {
    // Instance variables
    private String name;
    private int age;
    private double salary;

    public InstanceVariableDemo(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    public void display() {
        System.out.println(name + ", " + age + ", $" + salary);
    }

    public static void main(String[] args) {
        // Each instance has its own copy of instance variables
        InstanceVariableDemo alice = new InstanceVariableDemo("Alice", 30, 50000);
        InstanceVariableDemo bob = new InstanceVariableDemo("Bob", 25, 45000);

        alice.display();  // Alice, 30, $50000.0
        bob.display();    // Bob, 25, $45000.0

        // Changing alice's state doesn't affect bob
        alice.name = "Alice Smith";
        alice.display();  // Alice Smith, 30, $50000.0
        bob.display();    // Bob, 25, $45000.0
    }
}
```

**Expected Output:**
```
Alice, 30, $50000.0
Bob, 25, $45000.0
Alice Smith, 30, $50000.0
Bob, 25, $45000.0
```

### Real-World Cases

- **Entity Objects:** Representing database rows.
- **Value Objects:** Representing domain concepts (Money, Date).
- **GUI Components:** Storing component state (text, size, position).
- **Game Objects:** Storing position, health, and score.

---

## 7. Class Variables (Static Variables)

### Definitions

**Core Definition:** Class variables (static fields) are variables declared with the `static` keyword, shared by all instances of a class.

**Technical Definition:** "Class Variables (Static Fields): A class variable is any field declared with the `static` modifier; this tells the compiler that there is exactly one copy of this variable in existence, regardless of how many times the class has been instantiated. A `static` field is also known as a class variable because it is shared by all instances of the class". "Static variables are initialized when the class is loaded and are accessible without creating an instance of the class".

**Beginner-Friendly Explanation:** Static variables are like shared equipment in an office. No matter how many employees (objects) there are, there's only one coffee machine (static variable). All employees share it, and if one changes it, everyone sees the change.

### Purposes

- To store values shared by all instances of a class
- To define constants (with `static final`)
- To maintain class-level state (e.g., counters)
- To implement the Singleton pattern
- To provide utility methods and constants

### Syntax Rules and Structure

```java
public class MyClass {
    // Static variable
    private static int count;
    private static final double PI = 3.14159;
}
```

**Syntax Rules:**

- Declared with the `static` keyword.
- Shared by all instances of the class.
- Initialised when the class is loaded.
- Accessed via `ClassName.variableName` or (less preferred) `instanceName.variableName`.
- Automatically initialised to default values.

### Annotated Code Example

```java
// File: StaticVariableDemo.java
public class StaticVariableDemo {
    // Static variable: shared by all instances
    private static int instanceCount = 0;

    // Static constant
    public static final String APP_NAME = "MyApp";

    // Instance variable
    private String name;

    public StaticVariableDemo(String name) {
        this.name = name;
        instanceCount++;  // Increment shared counter
    }

    public static void main(String[] args) {
        StaticVariableDemo a = new StaticVariableDemo("A");
        StaticVariableDemo b = new StaticVariableDemo("B");
        StaticVariableDemo c = new StaticVariableDemo("C");

        // Access via class name (preferred)
        System.out.println("Instances: " + StaticVariableDemo.instanceCount);  // 3
        System.out.println("App: " + StaticVariableDemo.APP_NAME);  // MyApp

        // Access via instance (works but discouraged)
        System.out.println("Via instance: " + a.instanceCount);  // 3
    }
}
```

**Expected Output:**
```
Instances: 3
App: MyApp
Via instance: 3
```

### Real-World Cases

- **Constants:** `Math.PI`, `Integer.MAX_VALUE`.
- **Counters:** Tracking the number of instances created.
- **Configuration:** Application-wide settings.
- **Singleton:** `private static Singleton instance`.

---

## 8. Constants and `final` Keyword

### Definitions

**Core Definition:** A constant is a variable whose value cannot be changed after initialisation. In Java, constants are declared using the `final` keyword.

**Technical Definition:** "The `final` keyword is used to declare a variable that cannot be reassigned. A `final` variable must be initialised exactly once. `static final` variables are compile-time constants if the initialiser is a constant expression". "A `final` variable can be assigned only once. If it's a reference variable, the reference cannot be changed, but the object's contents can".

**Beginner-Friendly Explanation:** A `final` variable is like a permanent marker — once you write the value, you can't erase it. If it's a `static final` variable, it's a true constant shared by everyone and never changes.

### Purposes

- To define constants that should never change
- To prevent accidental reassignment
- To improve code readability (named constants instead of magic numbers)
- To enable compiler optimisations
- To enforce immutability for references

### Syntax Rules and Structure

```java
// Instance final
private final int id;

// Static final (constant)
public static final double PI = 3.14159;

// Local final
final int x = 10;
```

**Syntax Rules:**

- `final` variables must be initialised exactly once.
- `static final` variables are class-level constants.
- `final` reference variables cannot be reassigned, but the referenced object's contents can be modified.
- `final` parameters cannot be reassigned inside the method.

**Naming Convention:**

- `static final` constants: `UPPER_SNAKE_CASE` (e.g., `MAX_VALUE`, `PI`).
- `final` instance/local variables: camelCase.

### Annotated Code Example

```java
// File: FinalDemo.java
public class FinalDemo {
    // Static final constant
    public static final double TAX_RATE = 0.08;

    // Final instance variable (initialized in constructor)
    private final String id;

    // Final local variable
    public void method() {
        final int localConstant = 100;
        // localConstant = 200;  // Compile error
        System.out.println("Local: " + localConstant);
    }

    public FinalDemo(String id) {
        this.id = id;  // Must be initialized in constructor
    }

    public static void main(String[] args) {
        FinalDemo demo = new FinalDemo("ABC123");
        System.out.println("ID: " + demo.id);           // ABC123
        System.out.println("Tax: " + TAX_RATE);          // 0.08
        demo.method();                                   // Local: 100

        // Final reference: reference cannot change, object can
        final StringBuilder sb = new StringBuilder("Hello");
        sb.append(" World");  // OK — modifying the object
        System.out.println(sb);  // Hello World
        // sb = new StringBuilder();  // Compile error — cannot reassign
    }
}
```

**Expected Output:**
```
ID: ABC123
Tax: 0.08
Local: 100
Hello World
```

### Real-World Cases

- **Mathematical Constants:** `Math.PI`, `Math.E`.
- **Configuration Values:** `MAX_CONNECTIONS`, `DEFAULT_TIMEOUT`.
- **Error Codes:** `ERROR_NOT_FOUND = 404`.
- **Immutability:** Final fields in immutable classes.

---

## 9. `var` Keyword and Java 10 Type Inference

### Definitions

**Core Definition:** `var` is a reserved type name introduced in Java 10 that allows local variable type inference, letting the compiler infer the type from the initialiser.

**Technical Definition:** "Local variable type inference is a Java 10 feature that allows you to declare local variables without explicitly specifying their type. The compiler infers the type from the initialiser expression. The `var` keyword is not a keyword; it's a reserved type name, meaning it can still be used as a variable name, method name, or package name". "`var` can only be used for local variables, not for fields, method parameters, or return types".

**Beginner-Friendly Explanation:** `var` is like telling the compiler "you figure out the type — I'll just show you the value." Instead of writing `String name = "Alice";`, you write `var name = "Alice";` and the compiler sees the string and knows `name` is a `String`.

### Purposes

- To reduce boilerplate in local variable declarations
- To improve readability when the type is obvious from the initialiser
- To simplify code with complex generic types
- To keep variable declarations concise

### Syntax Rules and Structure

```java
// Instead of:
Map<String, List<String>> map = new HashMap<>();

// You can write:
var map = new HashMap<String, List<String>>();
```

**Syntax Rules:**

- `var` can only be used for local variables with an initialiser.
- `var` cannot be used for fields, method parameters, or return types.
- `var` cannot be used without an initialiser.
- `var` cannot be assigned `null` without a type.
- `var` is not a keyword; it's a reserved type name.

**Constraints and Limitations:**

- "`var` cannot be used for lambda parameters without explicit types".
- "`var` cannot be used in a catch block".
- "Excessive use of `var` can reduce readability if the type is not obvious".

### Annotated Code Example

```java
// File: VarDemo.java
import java.util.*;

public class VarDemo {
    public static void main(String[] args) {
        // Simple type inference
        var name = "Alice";           // String
        var age = 30;                  // int
        var salary = 50000.0;          // double
        var active = true;             // boolean

        System.out.println(name + ", " + age + ", $" + salary + ", " + active);

        // Complex generic types
        var list = new ArrayList<String>();
        list.add("Hello");
        list.add("World");
        System.out.println(list);  // [Hello, World]

        var map = new HashMap<String, List<Integer>>();
        map.put("evens", List.of(2, 4, 6));
        map.put("odds", List.of(1, 3, 5));
        System.out.println(map);  // {evens=[2, 4, 6], odds=[1, 3, 5]}

        // In loops
        for (var i = 0; i < 3; i++) {
            System.out.print(i + " ");
        }
        System.out.println();

        // With enhanced for
        var names = List.of("Alice", "Bob", "Charlie");
        for (var n : names) {
            System.out.print(n + " ");
        }
        System.out.println();
    }
}
```

**Expected Output:**
```
Alice, 30, $50000.0, true
[Hello, World]
{evens=[2, 4, 6], odds=[1, 3, 5]}
0 1 2 
Alice Bob Charlie 
```

### Real-World Cases

- **Collections:** `var list = new ArrayList<String>();`
- **Streams:** `var stream = list.stream();`
- **Try-with-Resources:** `try (var reader = Files.newBufferedReader(path))`.
- **Complex Generics:** `var map = new HashMap<String, List<String>>();`

---

## 10. Variable Scope and Lifetime

### Definitions

**Core Definition:** Scope is the region of code where a variable is accessible; lifetime is the period during which the variable exists in memory.

**Technical Definition:** "The scope of a variable determines where in the program the variable can be accessed. The lifetime of a variable determines how long the variable exists in memory". Local variables have block scope and method lifetime; instance variables have class scope and object lifetime; static variables have class scope and program lifetime.

**Beginner-Friendly Explanation:** Scope is like the visibility of a variable — who can see it. Lifetime is how long it exists. A local variable in a method is like a note written on a whiteboard — it's only visible in that room (scope) and gets erased when the meeting ends (lifetime).

### Scope and Lifetime Table

| Variable Type | Scope | Lifetime |
|---------------|-------|----------|
| Local | Block/method | Method/block execution |
| Instance | Class | Object lifetime |
| Static | Class | Program lifetime |
| Parameter | Method | Method execution |

### Annotated Code Example

```java
// File: ScopeLifetimeDemo.java
public class ScopeLifetimeDemo {
    // Instance variable: class scope, object lifetime
    private int instanceVar = 10;

    // Static variable: class scope, program lifetime
    private static int staticVar = 20;

    public void method() {
        // Local variable: method scope, method lifetime
        int localVar = 30;

        // Block scope
        if (true) {
            int blockVar = 40;
            System.out.println("Block: " + blockVar);  // 40
            System.out.println("Access outer: " + localVar);  // 30
        }
        // blockVar is out of scope here

        System.out.println("Instance: " + instanceVar);  // 10
        System.out.println("Static: " + staticVar);      // 20
    }

    public static void main(String[] args) {
        ScopeLifetimeDemo demo = new ScopeLifetimeDemo();
        demo.method();

        // Static accessible via class
        System.out.println("Static: " + ScopeLifetimeDemo.staticVar);  // 20
    }
}
```

**Expected Output:**
```
Block: 40
Access outer: 30
Instance: 10
Static: 20
Static: 20
```

### Real-World Cases

- **Encapsulation:** Limiting variable scope to prevent unintended access.
- **Memory Management:** Understanding when objects become eligible for GC.
- **Thread Safety:** Local variables are thread-safe; instance/static may need synchronization.

---

## 11. Variable Shadowing

### Definitions

**Core Definition:** Variable shadowing occurs when a variable in an inner scope has the same name as a variable in an outer scope, hiding the outer variable.

**Technical Definition:** "Shadowing occurs when a variable declared in an inner scope has the same name as a variable declared in an outer scope. The inner variable shadows the outer variable, making the outer variable inaccessible within the inner scope". In Java, local variables can shadow instance variables, and method parameters can shadow instance variables.

**Beginner-Friendly Explanation:** Shadowing is like having two people named "Alex" in the same office. When you call "Alex", the person closest to you answers. The outer Alex is still there, but you can't reach them by name from inside the inner room.

### Purposes

- (Often unintentional — a source of bugs)
- To allow parameter names to match field names (using `this` to disambiguate)
- To reuse common variable names in different scopes

### Syntax Rules and Structure

```java
public class ShadowDemo {
    private int value = 10;  // Instance variable

    public void method(int value) {  // Parameter shadows instance variable
        System.out.println(value);         // Parameter
        System.out.println(this.value);    // Instance variable
    }
}
```

**Syntax Rules:**

- Local variables can shadow instance variables.
- Method parameters can shadow instance variables.
- Local variables cannot shadow other local variables in the same scope.
- Use `this.fieldName` to access a shadowed instance variable.
- Use `ClassName.staticField` to access a shadowed static variable.

### Annotated Code Example

```java
// File: ShadowingDemo.java
public class ShadowingDemo {
    private int value = 100;           // Instance variable
    private static int count = 0;      // Static variable

    public void method(int value) {     // Parameter shadows instance variable
        System.out.println("Parameter: " + value);        // 50
        System.out.println("Instance: " + this.value);    // 100
        System.out.println("Static: " + ShadowingDemo.count);  // 0

        int localVar = value * 2;       // Local variable
        System.out.println("Local: " + localVar);          // 100

        // Block shadowing
        {
            int value2 = 200;            // Different name, no shadowing
            System.out.println("Block: " + value2);         // 200
        }
    }

    public static void main(String[] args) {
        new ShadowingDemo().method(50);
    }
}
```

**Expected Output:**
```
Parameter: 50
Instance: 100
Static: 0
Local: 100
Block: 200
```

### Real-World Cases

- **Constructor Parameters:** `this.name = name;` to distinguish parameter from field.
- **Setter Methods:** Same pattern as constructors.
- **Nested Scopes:** Avoiding accidental shadowing in loops and blocks.

---

## 12. Variable Naming### Definitions

**Core Definition:** Variable naming refers to the conventions and rules for choosing identifiers for variables.

**Technical Definition:** "Variable names must be valid Java identifiers: they must begin with a letter, underscore, or dollar sign, and can contain letters, digits, underscores, and dollar signs. By convention, variable names use camelCase (e.g., `firstName`, `totalCount`). Constants use UPPER_SNAKE_CASE (e.g., `MAX_VALUE`). Variable names should be descriptive and meaningful".

**Beginner-Friendly Explanation:** Variable naming is like choosing good labels for your boxes. A label like "x" doesn't tell you anything, but "customerAge" tells you exactly what's inside. Good names make code self-documenting.

### Naming Conventions Table

| Variable Type | Convention | Example |
|---------------|-----------|---------|
| Local variables | camelCase | `firstName`, `totalCount` |
| Instance variables | camelCase | `name`, `balance` |
| Static variables | camelCase | `instanceCount` |
| Constants (`static final`) | UPPER_SNAKE_CASE | `MAX_VALUE`, `PI` |
| Parameters | camelCase | `userName`, `orderId` |
| Type parameters | Single uppercase | `T`, `E`, `K`, `V` |

### Syntax Rules and Best Practices

- Use descriptive names that convey meaning.
- Avoid single-letter names (except loop counters `i`, `j`, `k`).
- Avoid abbreviations and acronyms.
- Avoid names that differ only in case.
- Use `is`/`has` prefixes for booleans (e.g., `isActive`, `hasPermission`).
- Do not use `$` in variable names (reserved for generated code).
- Avoid Java reserved keywords.

### Annotated Code Example

```java
// File: NamingDemo.java
public class NamingDemo {
    // GOOD: Descriptive names
    private String customerName;
    private int orderCount;
    private boolean isActive;
    private static final int MAX_RETRY_ATTEMPTS = 3;

    // BAD: Non-descriptive names (for illustration)
    // private String cn;
    // private int oc;
    // private boolean f;

    public void processOrder(String customerName, int orderId) {
        // GOOD: Meaningful local variables
        double orderTotal = 0.0;
        int itemCount = 0;

        for (int i = 0; i < 5; i++) {
            itemCount++;
            orderTotal += i * 10.0;
        }

        System.out.println("Customer: " + customerName);
        System.out.println("Order ID: " + orderId);
        System.out.println("Items: " + itemCount);
        System.out.println("Total: $" + orderTotal);
    }

    public static void main(String[] args) {
        NamingDemo demo = new NamingDemo();
        demo.processOrder("Alice", 1001);
    }
}
```

**Expected Output:**
```
Customer: Alice
Order ID: 1001
Items: 5
Total: $100.0
```

### Real-World Cases

- **API Design:** Public field names become part of the API.
- **Code Reviews:** Good names reduce the need for comments.
- **Maintenance:** Descriptive names make code easier to understand and modify.

---

## 13. Variable Best Practices

### Definitions

**Core Definition:** Variable best practices are guidelines for declaring, initialising, using, and naming variables to write clean, maintainable, and bug-free code.

**Technical Definition:** Best practices derive from authoritative sources including *Effective Java* by Joshua Bloch and the Java Language Specification. Key principles include declaring variables as close to their use as possible, minimising scope, preferring immutability with `final`, using meaningful names, and avoiding magic numbers.

**Beginner-Friendly Explanation:** Best practices help you avoid common mistakes like using confusing names, declaring variables too early, or forgetting to initialise them. They're like a checklist for writing clean, professional code.

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| Declare Close to Use | Declare variables as close to their first use as possible. |
| Minimise Scope | Keep variable scope as small as possible. |
| Prefer `final` | Use `final` for variables that don't need to change. |
| Initialise at Declaration | Initialise variables when you declare them whenever possible. |
| Use Descriptive Names | Choose names that clearly convey the variable's purpose. |
| Avoid Magic Numbers | Use named constants instead of hard-coded values. |
| Use `var` Judiciously | Use `var` when the type is obvious from the initialiser. |
| Avoid Shadowing | Don't reuse names in nested scopes; it causes confusion. |
| One Variable Per Declaration | Declare one variable per line for clarity. |
| Use `this` for Fields | Use `this.fieldName` to distinguish fields from parameters. |
| Prefer Local Variables | Prefer local variables over instance/static when possible. |
| Avoid Global Mutable State | Minimise static mutable variables. |

### Annotated Code Example

```java
// File: VariableBestPractices.java
public class VariableBestPractices {
    // BEST PRACTICE: Named constant instead of magic number
    private static final double TAX_RATE = 0.08;
    private static final int MAX_ITEMS = 100;

    // BEST PRACTICE: Descriptive instance variable names
    private final String customerName;
    private int itemCount;

    public VariableBestPractices(String customerName) {
        // BEST PRACTICE: Use 'this' to distinguish field from parameter
        this.customerName = customerName;
        this.itemCount = 0;
    }

    public double calculateTotal(double subtotal) {
        // BEST PRACTICE: Declare close to use, use final for constants
        final double tax = subtotal * TAX_RATE;
        final double total = subtotal + tax;
        return total;
    }

    public static void main(String[] args) {
        // BEST PRACTICE: Initialise at declaration
        var demo = new VariableBestPractices("Alice");
        double subtotal = 100.0;

        // BEST PRACTICE: Meaningful variable names
        double totalWithTax = demo.calculateTotal(subtotal);

        System.out.println("Customer: " + demo.customerName);
        System.out.println("Subtotal: $" + subtotal);
        System.out.println("Total: $" + totalWithTax);
    }
}
```

**Expected Output:**
```
Customer: Alice
Subtotal: $100.0
Total: $108.0
```

### Real-World Cases

- **Enterprise Applications:** Consistent naming and scoping conventions across teams.
- **Library Design:** Public constants with descriptive names.
- **Code Reviews:** Best practices form the basis for review checklists.
- **Static Analysis:** Tools like Checkstyle and PMD enforce best practices.

---

## References Links

- Variables — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/variables.html
- Primitive Data Types — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html
- Local Variable Type Inference — Oracle Java 10 Documentation - https://docs.oracle.com/javase/10/language/local-variable-type-inference.html
- var Keyword in Java — Baeldung - https://www.baeldung.com/java-var-keyword
- Java Language Specification, Chapter 4: Types, Values, and Variables - https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html
- Java Language Specification, Chapter 6: Names - https://docs.oracle.com/javase/specs/jls/se21/html/jls-6.html
- Java Code Conventions — Oracle - https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html
- Effective Java (3rd Edition) by Joshua Bloch — Chapter 2: Creating and Destroying Objects, Chapter 4: Classes and Interfaces
- Java Variables — W3Schools - https://www.w3schools.com/java/java_variables.asp
- Java Variables — GeeksforGeeks - https://www.geeksforgeeks.org/variables-in-java/