# Java Variable Scope: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Variable scope** in Java refers to the region of a program where a declared variable is visible, accessible, and valid. It determines from which parts of the code a variable can be referenced by its simple name.

### Technical Definition

The scope of a declaration is the region of the program within which the entity declared by the declaration can be referred to using a simple name. In Java, scope is determined by the **block structure** of the program: a variable declared in a block is in scope from its declaration to the end of that block, including any nested blocks . Java distinguishes several scopes: **local scope** (variables declared in method bodies), **block scope** (variables declared in nested blocks such as `if`, `for`, or `while`), **parameter scope** (formal parameters of methods, constructors, and catch clauses), **instance scope** (instance variables visible throughout the class and accessible via `this`), and **static scope** (class variables shared by all instances). Variables with **effectively final** status are those that are never reassigned after initialization, allowing them to be captured by local inner classes and lambda expressions .

### Beginner-Friendly Explanation

Variable scope is like the range of a walkie-talkie. A walkie-talkie in your room (a local variable) only works inside that room. A walkie-talkie in the living room (an instance variable) works throughout the house. A walkie-talkie connected to a central system (a static variable) works everywhere, even for visitors. If you try to use a walkie-talkie outside its range, you get static—or in Java, a compile-time error.

### Key Characteristics

- **Block-based**: Scope is determined by the curly braces `{ }` that enclose a declaration.
- **Lexical**: Scope is determined by the structure of the source code, not by runtime execution order.
- **Hierarchical**: Inner blocks can access variables from outer blocks, but not vice versa.
- **Shadowing**: A local variable with the same name as a field hides the field within its scope; `this.field` disambiguates.
- **Lifetime vs. scope**: Scope is about visibility; lifetime is about when the variable exists in memory.
- **Effectively final**: A variable that is never reassigned can be captured by lambdas and local inner classes.

### Prerequisites

- Basic Java syntax (variables, methods, classes).
- Understanding of blocks and control flow (`if`, `for`, `while`).
- Familiarity with instance vs. static members.

### Related Programming Areas

- **Encapsulation**: Access modifiers control visibility beyond scope.
- **Lambda Expressions**: Require effectively final variables for capture.
- **Memory Management**: Scope influences when variables become garbage.
- **Concurrency**: Shared mutable state requires careful scope management.

### Core Concepts / Features

1. Local Scope
2. Block Scope
3. Parameter Scope
4. Instance Scope (`this` keyword and shadowing)
5. Static Scope (Class-Level Variables)
6. Effectively Final Variables


## Core Concept 1: Local Scope

### Definitions

**Core Definition**: **Local scope** refers to variables declared inside a method body (or constructor, or initializer block) that are visible from their declaration to the end of the enclosing block.

**Technical Definition**: A local variable is a variable declared within a method's body, a constructor's body, or a static/instance initializer block. The scope of a local variable declared in a block by a local variable declaration statement is the rest of the block, starting with the declaration's own initializer and including any further declarators to the right in the same declaration statement . Local variables are not accessible from outside their method and do not have default values; they must be explicitly initialized before use.

**Beginner-Friendly Explanation**: Local variables are like notes you write on a whiteboard in a meeting room. They only exist during the meeting (method execution) and only in that room. When the meeting ends, the notes are erased. No one outside the room can see them.

### Purposes

- To store temporary data needed only during a method's execution.
- To avoid polluting the class namespace with variables that have no meaning outside the method.
- To enable recursion by giving each method call its own copy of local variables.
- To promote encapsulation by keeping implementation details local.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public ReturnType methodName(parameters) {
    // Local variable declaration
    Type localVar = initialValue;
    // ...
    // localVar is in scope from declaration to end of method body
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Type` | The data type of the local variable. |
| `localVar` | The variable identifier. |
| `initialValue` | The initial value (mandatory before use). |

**Syntax Rules:**

- Local variables must be initialized before they are used.
- Local variables cannot have access modifiers (`public`, `private`, etc.).
- The scope of a local variable starts at its declaration and ends at the closing brace of the enclosing block.
- Multiple variables can be declared in one statement: `int a = 1, b = 2;`.
- Local variables cannot be declared `static`.

**Constraints and Limitations:**

- Local variables are destroyed when the method returns.
- They cannot be accessed from outside the method.
- Uninitialized local variables cause compile-time errors.
- Local variables cannot be shadowed by other local variables in the same scope.

### Annotated Complete Code Examples

**Example 1: Local Variable Scope**

```java
/**
 * Demonstrates local variable scope within a method.
 */
public class LocalScopeDemo {

    public static void calculate() {
        // Local variable: in scope from here to end of method
        int result = 10;

        System.out.println("Result: " + result);

        // Another local variable
        String message = "Done";
        System.out.println(message);

        // 'result' and 'message' are both accessible here
    }

    public static void main(String[] args) {
        calculate();
        // System.out.println(result); // Compile error: result is not in scope
    }
}
```

**Expected Output:**

```
Result: 10
Done
```

**Why This Output Occurs:**
- `result` and `message` are declared inside `calculate()` and are only visible within that method.
- Attempting to access `result` from `main()` would cause a compile-time error because it is out of scope.

### Real-World Cases

- **Loop counters**: `for (int i = 0; ...)` — `i` is local to the loop.
- **Temporary calculations**: Intermediate results in algorithms.
- **String building**: `StringBuilder sb = new StringBuilder()` inside a method.
- **Exception handling**: Variables declared inside `try` or `catch` blocks.

### References

- Java Language Specification – Scope of a Local Variable Declaration - https://docs.oracle.com/javase/specs/jls/se17/html/jls-6.html#jls-6.3
- University of Kentucky – Java Variables and Arrays (Scope Rules) - https://cs.uky.edu/~cheng/cs335/Notes/Java06-2-var.pdf
- UCSD CSE 11 – Variable Visibility - https://cseweb.ucsd.edu/~kube/cls/11.s11/Lectures/lec5/lec5.pdf


## Core Concept 2: Block Scope

### Definitions

**Core Definition**: **Block scope** refers to variables declared inside a nested block (such as an `if`, `for`, `while`, or explicit `{ }` block) that are visible only within that block.

**Technical Definition**: A block is a sequence of statements enclosed in braces `{ }`. The scope of a local variable declared in a block is the rest of that block, starting with the declaration's own initializer . Variables declared in a `for` loop's initialization are scoped to the loop's expression, update, and body. Variables declared in an enhanced `for` statement's header are scoped to the contained statement. Variables declared in a `try-with-resources` resource specification are in scope from the declaration rightward over the remainder of the resource specification and the entire `try` block .

**Beginner-Friendly Explanation**: Block scope is like having separate drawers in a filing cabinet. A variable declared in the "if" drawer is only accessible while you're working in that drawer. Once you close it, the variable is gone. You can have a variable with the same name in a different drawer without conflict.

### Purposes

- To limit the visibility of temporary variables to the smallest necessary region.
- To enable variable name reuse in different blocks without conflict.
- To prevent accidental access to variables outside their intended context.
- To support loop counters that don't leak beyond the loop.

### Syntax Rules and Structure

**Complete General Syntax (Explicit Block):**

```java
{
    Type blockVar = value;
    // blockVar in scope here
}
// blockVar out of scope here
```

**Complete General Syntax (If Block):**

```java
if (condition) {
    Type ifVar = value;
    // ifVar in scope here
}
// ifVar out of scope here
```

**Complete General Syntax (For Loop):**

```java
for (int i = 0; i < n; i++) {
    // i is in scope here
}
// i is out of scope here
```

**Syntax Rules:**

- Variables declared in a block are only visible within that block.
- Inner blocks can access variables from outer blocks (lexical scoping).
- A variable declared in an inner block can shadow a variable from an outer block.
- Loop variables are scoped to the loop body and are not accessible after the loop.

**Constraints and Limitations:**

- Cannot access a block-scoped variable outside its block.
- Shadowing can make code confusing; use distinct names for clarity.
- Variables in `for` loop headers are re-created for each iteration (in enhanced for).

### Annotated Complete Code Examples

**Example 1: Block Scope with If and For**

```java
/**
 * Demonstrates block scope in if statements and for loops.
 */
public class BlockScopeDemo {
    public static void main(String[] args) {
        int outerVar = 100;

        // If block scope
        if (outerVar > 50) {
            int innerVar = 200;
            System.out.println("Inside if: outerVar = " + outerVar);
            System.out.println("Inside if: innerVar = " + innerVar);
        }
        // System.out.println(innerVar); // Compile error: out of scope

        // For loop scope
        for (int i = 0; i < 3; i++) {
            System.out.println("Loop iteration: i = " + i);
        }
        // System.out.println(i); // Compile error: i out of scope

        // Nested blocks and shadowing
        int x = 10;
        {
            int xShadow = 20; // Different variable, no conflict
            System.out.println("Inner block: xShadow = " + xShadow);
        }
        System.out.println("Outer: x = " + x);
    }
}
```

**Expected Output:**

```
Inside if: outerVar = 100
Inside if: innerVar = 200
Loop iteration: i = 0
Loop iteration: i = 1
Loop iteration: i = 2
Inner block: xShadow = 20
Outer: x = 10
```

**Why This Output Occurs:**
- `innerVar` is declared inside the `if` block and is only accessible there.
- `i` is declared in the `for` loop header and is only accessible in the loop body.
- `xShadow` is in a nested block and does not conflict with `x` (different name).

### Real-World Cases

- **Loop counters**: `for (int i = 0; i < list.size(); i++)`.
- **Conditional temporaries**: Variables used only inside an `if` branch.
- **Try-with-resources**: Resources scoped to the try block.
- **Switch statements**: Variables declared in case blocks.

### References

- Java Language Specification – Scope of a Local Variable Declaration - https://docs.oracle.com/javase/specs/jls/se17/html/jls-6.html#jls-6.3
- Java Language Specification – Blocks - https://docs.oracle.com/javase/specs/jls/se17/html/jls-14.html#jls-14.2
- UCSD CSE 11 – Variable Visibility - https://cseweb.ucsd.edu/~kube/cls/11.s11/Lectures/lec5/lec5.pdf


## Core Concept 3: Parameter Scope

### Definitions

**Core Definition**: **Parameter scope** refers to the visibility of a method's formal parameters, which are visible only within the method body.

**Technical Definition**: Formal parameters (also called formal arguments) are identifiers in the argument list of a method or constructor definition. They act like variables that are visible only in the method body . The scope of a parameter of an exception handler declared in a `catch` clause is the entire block associated with the catch . Parameters are initialized with the values of the actual arguments when the method is called; they are distinct variables from the caller's variables, even if the values are the same.

**Beginner-Friendly Explanation**: Parameters are like ingredients handed to a chef. The chef can use those ingredients only while cooking that dish. Once the dish is done, the ingredients are gone. The chef cannot access ingredients from a different recipe.

### Purposes

- To pass data into a method for processing.
- To provide named, local variables for the method's arguments.
- To isolate each method call's arguments from other calls.
- To enable recursion by giving each call its own parameter values.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public ReturnType methodName(Type1 param1, Type2 param2) {
    // param1 and param2 are in scope here
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Type1 param1` | The first formal parameter. |
| `Type2 param2` | The second formal parameter. |

**Syntax Rules:**

- Parameters are visible throughout the method body.
- Parameters cannot be accessed from outside the method.
- A parameter can shadow a field with the same name; use `this.field` to access the field.
- Parameters are passed by value; the parameter is a copy of the argument.
- Catch block parameters are scoped to the catch block.

**Constraints and Limitations:**

- Parameter names must be unique within the method's parameter list.
- Parameters cannot be declared `static`.
- Modifying a parameter does not affect the caller's argument (pass-by-value).
- Catch parameters cannot be reassigned in the catch block (they are implicitly final in Java 7+? Actually, they can be reassigned but it's discouraged).

### Annotated Complete Code Examples

**Example 1: Parameter Scope**

```java
/**
 * Demonstrates parameter scope within a method.
 */
public class ParameterScopeDemo {

    // Parameter 'name' is in scope only within this method
    public static void greet(String name) {
        System.out.println("Hello, " + name + "!");
        // 'name' is accessible here
    }

    // Multiple parameters
    public static int add(int a, int b) {
        // Both 'a' and 'b' are in scope
        int sum = a + b;
        return sum;
    }

    public static void main(String[] args) {
        greet("Alice");
        // System.out.println(name); // Compile error: name not in scope

        int result = add(5, 3);
        System.out.println("Sum: " + result);
        // System.out.println(a); // Compile error: a not in scope
    }
}
```

**Expected Output:**

```
Hello, Alice!
Sum: 8
```

**Why This Output Occurs:**
- `name` is a parameter of `greet()` and is only accessible inside that method.
- `a` and `b` are parameters of `add()` and are only accessible inside that method.
- From `main()`, neither `name` nor `a`/`b` can be accessed.

### Real-World Cases

- **Setter methods**: `setName(String name)` where the parameter shadows the field.
- **Constructors**: Parameters initialize instance fields.
- **Utility methods**: `Math.max(int a, int b)` uses parameters.
- **Exception handling**: `catch (IOException e)` where `e` is scoped to the catch block.

### References

- UCSD CSE 11 – Formal Arguments as Variables - https://cseweb.ucsd.edu/~kube/cls/11.s11/Lectures/lec5/lec5.pdf
- Java Language Specification – Scope of a Local Variable Declaration - https://docs.oracle.com/javase/specs/jls/se17/html/jls-6.html#jls-6.3
- Java Language Specification – Formal Parameters - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.4.1


## Core Concept 4: Instance Scope (`this` Keyword and Shadowing)

### Definitions

**Core Definition**: **Instance scope** refers to instance variables (fields) that belong to each object and are visible throughout the class, accessible via the object reference or the `this` keyword.

**Technical Definition**: An instance variable is a field declared in a class without the `static` modifier. It belongs to each instance of the class; at runtime, there are as many copies of the instance variable as there are instances . Instance variables are visible in every instance method of the class, unless hidden by a local variable or parameter with the same name (shadowing). The `this` keyword refers to the current object and can be used to disambiguate a field from a parameter or local variable with the same name.

**Beginner-Friendly Explanation**: Instance variables are like personal belongings in your house. Each house (object) has its own set. When you're inside the house, you can access your belongings directly. If someone gives you a new item with the same name (a parameter), you can say "the one in my house" (`this.field`) to distinguish it from the new item.

### Purposes

- To store the state of each object persistently across method calls.
- To allow methods to access and modify the object's state.
- To enable encapsulation by hiding state behind access modifiers.
- To provide a way to distinguish fields from parameters via `this`.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class ClassName {
    // Instance variable (field)
    private Type fieldName;

    public void methodName(Type fieldName) {
        // 'fieldName' refers to the parameter (shadowing)
        // 'this.fieldName' refers to the instance variable
        this.fieldName = fieldName;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `private Type fieldName` | Instance variable declaration. |
| `Type fieldName` (parameter) | Parameter that shadows the field. |
| `this.fieldName` | Refers to the instance variable. |
| `fieldName` | Refers to the parameter (shadowing). |

**Syntax Rules:**

- Instance variables are declared inside the class but outside any method.
- Instance variables are visible throughout the class (all instance methods).
- Instance variables are not visible in static methods without an object reference.
- `this` is available only in instance methods and constructors.
- Shadowing occurs when a local variable or parameter has the same name as a field.

**Constraints and Limitations:**

- Static methods cannot access instance variables directly (no `this`).
- Shadowing can make code confusing; use `this` for clarity.
- Instance variables have default values (`0`, `false`, `null`) if not initialized.
- Instance variables are created when the object is constructed and destroyed when garbage collected.

### Annotated Complete Code Examples

**Example 1: Instance Scope and Shadowing**

```java
/**
 * Demonstrates instance scope, shadowing, and the this keyword.
 */
public class Person {
    // Instance variables
    private String name;
    private int age;

    // Constructor: parameters shadow fields
    public Person(String name, int age) {
        // 'name' refers to the parameter; 'this.name' refers to the field
        this.name = name;
        this.age = age;
    }

    // Instance method: fields are visible
    public void introduce() {
        System.out.println("Hi, I'm " + name + " and I'm " + age + " years old.");
        // 'name' and 'age' refer to the fields (no local shadowing)
    }

    // Method with a local variable that shadows the field
    public void setName(String name) {
        String localName = "Local: " + name; // 'name' is the parameter
        this.name = localName; // 'this.name' is the field
    }

    public static void main(String[] args) {
        Person person = new Person("Alice", 30);
        person.introduce();

        person.setName("Bob");
        person.introduce();
    }
}
```

**Expected Output:**

```
Hi, I'm Alice and I'm 30 years old.
Hi, I'm Local: Bob and I'm 30 years old.
```

**Why This Output Occurs:**
- The constructor uses `this.name = name` to assign the parameter to the field.
- `introduce()` accesses the fields directly (no shadowing).
- `setName()` shadows the field with the parameter `name`, uses a local variable `localName`, and assigns via `this.name`.

### Real-World Cases

- **Entity classes**: `User`, `Product`, `Order` with fields and getters/setters.
- **Constructors**: Initializing fields from parameters.
- **Setters**: Modifying state with `this.field = field`.
- **Encapsulation**: Private fields with public accessors.

### References

- UCSD CSE 11 – Instance Variables - https://cseweb.ucsd.edu/~kube/cls/11.s11/Lectures/lec5/lec5.pdf
- University of Kentucky – Instance Variables - https://cs.uky.edu/~cheng/cs335/Notes/Java06-2-var.pdf
- Java Language Specification – Field Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3


## Core Concept 5: Static Scope (Class-Level Variables)

### Definitions

**Core Definition**: **Static scope** refers to static variables (class variables) that belong to the class itself, not to any individual instance, and are shared by all instances of the class.

**Technical Definition**: A static variable is a field declared with the `static` modifier. It exists once the class is loaded into memory, and its value persists until the program ends (unless explicitly changed). Static variables are shared among all objects of the same class and can be accessed using the class name (`ClassName.staticVariable`) . Static methods cannot access non-static (instance) variables directly because there is no `this` reference.

**Beginner-Friendly Explanation**: Static variables are like a shared whiteboard in an office. Everyone in the office can see and write on the same whiteboard. It doesn't belong to any one person—it belongs to the office itself. If you're working in a different office (static method), you can't see the whiteboard unless you go to the office (use the class name).

### Purposes

- To share data across all instances of a class.
- To declare constants (with `static final`).
- To implement counters, caches, or configuration values.
- To provide utility methods that don't depend on instance state.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class ClassName {
    // Static variable (class variable)
    private static Type staticField;

    // Static constant
    public static final Type CONSTANT = value;

    // Static method
    public static ReturnType staticMethod() {
        // Can access staticField directly
        // Cannot access instance fields directly
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `private static Type staticField` | Static variable declaration. |
| `public static final Type CONSTANT` | Static constant. |
| `ClassName.staticField` | Access from outside the class. |

**Syntax Rules:**

- Static variables are declared with the `static` modifier.
- Static variables are shared by all instances; only one copy exists.
- Static variables are initialized when the class is loaded.
- Static methods can access static variables and other static methods directly.
- Static methods cannot access instance variables or instance methods directly.
- Static variables can be accessed via `ClassName.staticVariable` or `instance.staticVariable` (discouraged).

**Constraints and Limitations:**

- Static variables live for the entire program lifetime (until JVM exits).
- Static methods cannot use `this` or `super`.
- Overuse of static variables can lead to global state and testing difficulties.
- Static variables are not thread-safe unless properly synchronized.

### Annotated Complete Code Examples

**Example 1: Static Scope and Shared State**

```java
/**
 * Demonstrates static variables and methods.
 */
public class Employee {
    // Instance variables: unique per object
    private String name;

    // Static variable: shared by all instances
    private static int employeeCount = 0;

    // Static constant
    public static final String COMPANY = "TechCorp";

    public Employee(String name) {
        this.name = name;
        employeeCount++; // Increment shared counter
    }

    public String getName() {
        return name;
    }

    // Static method: can access static variables, not instance variables
    public static int getEmployeeCount() {
        return employeeCount;
        // return name; // Compile error: cannot access instance variable
    }

    public static void main(String[] args) {
        System.out.println("Company: " + Employee.COMPANY);
        System.out.println("Count before: " + Employee.getEmployeeCount());

        Employee e1 = new Employee("Alice");
        Employee e2 = new Employee("Bob");

        System.out.println("e1: " + e1.getName());
        System.out.println("e2: " + e2.getName());
        System.out.println("Count after: " + Employee.getEmployeeCount());
    }
}
```

**Expected Output:**

```
Company: TechCorp
Count before: 0
e1: Alice
e2: Bob
Count after: 2
```

**Why This Output Occurs:**
- `employeeCount` is static and shared; it increments with each constructor call.
- `COMPANY` is a static constant accessible via `Employee.COMPANY`.
- `getEmployeeCount()` is static and returns the shared count.
- `e1.getName()` and `e2.getName()` access instance-specific `name` fields.

### Real-World Cases

- **Counters**: Tracking the number of instances created.
- **Constants**: `Math.PI`, `Integer.MAX_VALUE`.
- **Configuration**: `System.out`, `System.err`.
- **Utility methods**: `Math.sqrt()`, `Collections.sort()`.

### References

- University of Kentucky – Static Variables and Scope Rules - https://cs.uky.edu/~cheng/cs335/Notes/Java06-2-var.pdf
- Java Language Specification – Static Fields - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3.1.1
- UCSD CSE 11 – Static Variables - https://cseweb.ucsd.edu/~kube/cls/11.s11/Lectures/lec5/lec5.pdf


## Core Concept 6: Effectively Final Variables

### Definitions

**Core Definition**: An **effectively final** variable is a local variable or parameter whose value is never changed after it is initialized, even though it is not explicitly declared with the `final` keyword.

**Technical Definition**: A local variable or parameter that is never used as the left-hand operand of an assignment operator or as the operand of a pre-increment, pre-decrement, post-increment, or post-decrement operator is considered effectively final . Effectively final variables can be captured by local inner classes and lambda expressions, which previously required explicit `final` declarations. If a variable is effectively final, adding the `final` modifier to its declaration will not introduce any compile-time errors, and removing `final` from an explicitly final variable will not affect the program's validity .

**Beginner-Friendly Explanation**: An effectively final variable is like a promise you never break. You say, "I'll set this value once and never change it." Java recognizes this and lets you use the variable inside lambdas and inner classes, which need stable values. You don't have to write `final`—Java figures it out.

### Purposes

- To allow local inner classes and lambdas to capture local variables without explicit `final`.
- To reduce the verbosity of code that uses closures.
- To ensure thread safety by preventing mutation of captured variables.
- To express intent that a variable is immutable after initialization.

### Syntax Rules and Structure

**Complete General Syntax (Effectively Final):**

```java
// Effectively final: never reassigned
int count = 10;
Runnable r = () -> System.out.println(count); // OK

// NOT effectively final: reassigned
int total = 0;
total = total + 1; // Reassigned
// Runnable r2 = () -> System.out.println(total); // Compile error
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Variable declaration | A local variable or parameter. |
| No reassignment | The variable is never assigned after initialization. |
| Lambda/inner class | The variable can be captured. |

**Syntax Rules:**

- A variable is effectively final if it is never reassigned after initialization.
- Effectively final variables can be captured by lambdas and local inner classes.
- Explicitly declared `final` variables are always effectively final.
- The compiler infers effectively final status; no explicit keyword is needed.
- Parameters are effectively final if not reassigned within the method.

**Constraints and Limitations:**

- If a variable is reassigned, it cannot be captured by a lambda or inner class.
- Effectively final applies only to local variables and parameters, not fields.
- The rules for effectively final are determined by definite assignment analysis.

### Annotated Complete Code Examples

**Example 1: Effectively Final in Lambda**

```java
import java.util.*;

/**
 * Demonstrates effectively final variables in lambdas.
 */
public class EffectivelyFinalDemo {
    public static void main(String[] args) {
        // Effectively final: never reassigned
        String greeting = "Hello";

        // Lambda captures the effectively final variable
        Runnable r1 = () -> System.out.println(greeting);
        r1.run();

        // NOT effectively final: reassigned
        int counter = 0;
        // counter = 1; // Uncommenting makes it not effectively final
        // Runnable r2 = () -> System.out.println(counter); // Would be OK if not reassigned

        // Effectively final parameter
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        names.forEach(name -> System.out.println(greeting + ", " + name));
    }
}
```

**Expected Output:**

```
Hello
Hello, Alice
Hello, Bob
Hello, Charlie
```

**Why This Output Occurs:**
- `greeting` is never reassigned, so it is effectively final and can be captured by the lambda.
- The lambda passed to `forEach` captures `greeting` and the parameter `name`.
- The `counter` variable is declared but never used in a lambda in this example.

### Real-World Cases

- **Stream API**: Capturing configuration values in `filter`, `map`, `forEach`.
- **Event handlers**: Capturing UI state in listeners.
- **Concurrency**: Capturing immutable data in `Runnable` or `Callable`.
- **Functional interfaces**: Passing effectively final variables to `Predicate`, `Function`, etc.

### References

- Patent US9542166 – Effectively Final Variables - https://patentimages.storage.googleapis.com/33/43/aa/2f7c93add3f0ac/US9542166.pdf
- Java Language Specification – Effectively Final - https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.12.4
- Oracle Java Tutorials – Lambda Expressions - https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html


## References

- Java Language Specification – Scope of a Local Variable Declaration - https://docs.oracle.com/javase/specs/jls/se17/html/jls-6.html#jls-6.3
- Java Language Specification – Blocks - https://docs.oracle.com/javase/specs/jls/se17/html/jls-14.html#jls-14.2
- Java Language Specification – Formal Parameters - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.4.1
- Java Language Specification – Field Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3
- Java Language Specification – Static Fields - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3.1.1
- Java Language Specification – Effectively Final - https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.12.4
- University of Kentucky – Java Variables and Arrays (Scope Rules) - https://cs.uky.edu/~cheng/cs335/Notes/Java06-2-var.pdf
- UCSD CSE 11 – Variable Visibility - https://cseweb.ucsd.edu/~kube/cls/11.s11/Lectures/lec5/lec5.pdf
- Patent US9542166 – Effectively Final Variables - https://patentimages.storage.googleapis.com/33/43/aa/2f7c93add3f0ac/US9542166.pdf
- Oracle Java Tutorials – Lambda Expressions - https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html
- OpenJDK – Scope of a Local Variable Declaration - https://bugs.openjdk.org/secure/attachment/113767/Module%20Import%20Declarations%20%28Second%20Preview%29%20-%20JLS.pdf