# Java Methods: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

A **Java method** is a named, reusable block of code declared within a class that encapsulates a specific behavior. Methods define what an object can do or what a class can perform, acting as the behavioral counterpart to fields (which define what an object *is*).

### Technical Definition

A method is a member of a class declared with a return type, a name, a parameter list, an optional exception list, and a body enclosed in braces. Methods may be **instance methods** (invoked on objects, with access to instance state via `this`) or **static methods** (invoked on the class itself, without access to instance state). The **method signature** consists of the method's name and parameter types, which uniquely identify a method within its class and enable overloading.

### Beginner-Friendly Explanation

Think of a method as a recipe card in a cookbook (the class). Each card has a name (like `bakeCake`), a list of ingredients you need to provide (parameters), and instructions (the method body). Some recipes produce a finished dish you can eat (return a value), while others just do something like "preheat the oven" and don't give you anything back (`void`). You can follow the same recipe many times, and you can have multiple recipes with similar names as long as the ingredients differ.

### Key Characteristics

- **Reusability**: A method is declared once but can be invoked many times.
- **Encapsulation**: Methods bundle logic and expose only what is necessary through access modifiers.
- **Signature-based identification**: A method is uniquely identified by its name and parameter types, not its return type.
- **Two flavors**: Instance methods operate on object state; static methods operate at the class level.
- **Pass-by-value**: Java always passes arguments by value—primitives copy the value, objects copy the reference.

### Prerequisites

- Basic Java syntax (variables, data types, operators).
- Understanding of classes and objects.
- Familiarity with the `main` method and program entry point.

### Related Programming Areas

- **Object-Oriented Programming**: Methods are the "behavior" half of encapsulation.
- **API Design**: Public methods form the contract of a class.
- **Design Patterns**: Strategy, Template Method, and Command patterns rely on method design.
- **Functional Programming**: Lambdas and method references are concise method representations.

### Core Concepts / Features

1. Method Declaration
2. Method Invocation
3. Parameters and Arguments
4. Return Values
5. `void`
6. Method Signatures (Name and Parameter Types)
7. Method Scope
8. Access Modifiers and Visibility
9. Static vs. Instance Methods


## Core Concept 1: Method Declaration

### Definitions

**Core Definition**: A method declaration is the formal specification of a method, including its name, return type, parameters, and body, written inside a class.

**Technical Definition**: A method declaration consists of six components in order: modifiers, return type, method name, parameter list in parentheses, an exception list, and a method body in braces. The only required elements are the return type, name, parentheses, and body. A method declaration resides within a class definition and cannot be nested inside another method.

**Beginner-Friendly Explanation**: A method declaration is like writing down a recipe. You give it a name, list what ingredients it needs, say what it produces, and write the steps. This recipe card lives in the cookbook (class) and can be used whenever you need it.

### Purposes

- To define reusable behavior that can be invoked from other parts of the program.
- To encapsulate logic behind a meaningful name, improving readability.
- To establish a clear contract for how the method should be called (parameters and return type).
- To organize code into logical, testable units.

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
| `[access_modifier]` | `public`, `protected`, package-private (none), or `private`. |
| `[static]` | Indicates a class-level method (no instance required). |
| `[final]` | Prevents overriding in subclasses. |
| `[abstract]` | Declares a method without a body (must be overridden). |
| `ReturnType` | The type of value returned, or `void` if none. |
| `methodName` | Identifier following lowerCamelCase convention (verb-first). |
| `(ParameterList)` | Comma-separated type-name pairs; empty `()` if no parameters. |
| `[throws ExceptionList]` | Checked exceptions the method may throw. |
| `{ ... }` | The method body containing executable statements. |

**Syntax Rules:**

- The only required elements are the return type, name, `()`, and `{}`.
- Method names should be verbs in lowercase or multi-word names beginning with a verb (e.g., `calculateTotal`, `getBackground`).
- If there are no parameters, empty parentheses `()` are required.
- A method with a non-`void` return type must return a value on all code paths.
- Methods cannot be declared outside a class or inside another method.

**Constraints and Limitations:**

- `abstract` methods have no body and must be in abstract classes or interfaces.
- `final` methods cannot be overridden by subclasses.
- `static` methods cannot access instance fields or methods directly.
- You cannot declare two methods with the same name and same parameter types (regardless of return type).

### Annotated Complete Code Examples

**Example 1: Method declaration: no parameters, returns void**

```java
/**
 * A class demonstrating method declarations.
 */
public class Greeter {

    public void sayHello() {
        System.out.println("Hello, world!");
    }

    public static void main(String[] args) {
        Greeter greeter = new Greeter();
        
        // Invoking the methods
        greeter.sayHello();
    }
}
```

**Expected Output:**

```
Hello, world!
```

**Why This Output Occurs:**
- `sayHello()` is declared with `void` return type and no parameters; it prints a fixed message.
- Each method is declared once but invoked multiple times.

**Example 2: Method declaration: one parameter, returns String**

```java
/**
 * A class demonstrating method declarations.
 */
public class Greeter {
    public String greet(String name) {
        return "Hello, " + name + "!";
    }

    public static void main(String[] args) {
        Greeter greeter = new Greeter();
        
        System.out.println(greeter.greet("Alice"));
    }
}
```

**Expected Output:**

```
Hello, Alice!
```

**Why This Output Occurs:**
- `greet(String name)` takes a `String` parameter and returns a `String`.
- Each method is declared once but invoked multiple times.

**Example 3: Method declaration: multiple parameters, returns int**

```java
/**
 * A class demonstrating method declarations.
 */
public class Greeter {
    public int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        Greeter greeter = new Greeter();

        System.out.println("Sum: " + greeter.add(3, 4));
    }
}
```

**Expected Output:**

```
Sum: 7
```

**Why This Output Occurs:**
- `add(int a, int b)` takes two `int` parameters and returns their sum.
- Each method is declared once but invoked multiple times.

**Step-by-Step Setup Guide:**
1. Create `Greeter.java`.
2. Compile with `javac Greeter.java`.
3. Run with `java Greeter`.
4. Observe the output.

### Real-World Cases

- **Service classes**: `UserService.createUser()`, `OrderService.processOrder()`.
- **Utility classes**: `Math.max()`, `Collections.sort()`.
- **Domain models**: `BankAccount.deposit()`, `ShoppingCart.addItem()`.
- **API endpoints**: Controller methods handling HTTP requests.

### References

- Oracle Java Tutorials – Defining Methods - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html
- Java Language Specification – Method Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.4
- CMU – Methods Handout - http://www.cs.cmu.edu/afs/cs.cmu.edu/user/mrmiller/www/15-110/Handouts/methods-4.pdf


## Core Concept 2: Method Invocation

### Definitions

**Core Definition**: Method invocation (also called method call) is the process of executing a method by referencing its name and providing any required arguments.

**Technical Definition**: When a method is invoked, program execution transfers to that method, the method body is executed, and when the method finishes, execution returns to the place from where the method was called. Instance methods are invoked via an object reference (`object.method()`), while static methods are invoked via the class name (`ClassName.method()`). A method can have only a single declaration but can be called any number of times.

**Beginner-Friendly Explanation**: Invoking a method is like pressing a button. The button has a label (method name), and when you press it, something happens. You can press the same button as many times as you want.

### Purposes

- To execute the behavior defined by a method.
- To reuse code without duplicating it.
- To delegate work from one method to another.
- To trigger side effects (printing, modifying state) or obtain return values.

### Syntax Rules and Structure

**Complete General Syntax (Instance Method):**

```java
objectReference.methodName(arguments);
```

**Complete General Syntax (Static Method):**

```java
ClassName.methodName(arguments);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `objectReference` | The object on which the instance method is invoked. |
| `ClassName` | The class name for static method invocation. |
| `.` | The member access operator. |
| `methodName` | The name of the method to invoke. |
| `(arguments)` | Comma-separated values matching the method's parameters. |

**Syntax Rules:**

- Instance methods require an object reference; static methods use the class name.
- The arguments must match the method's parameters in type, order, and number.
- Method invocation can be part of an expression if the method returns a value.
- A `void` method invocation is a statement, not an expression.

**Constraints and Limitations:**

- Invoking a method on a `null` reference throws `NullPointerException`.
- The arguments provided must be assignable to the parameter types.
- Static methods cannot be invoked on an instance reference (though Java allows it with a warning).

### Annotated Complete Code Examples

**Example 1: Instance vs. Static Method Invocation**

```java
public class Calculator {

    // Instance method
    private int memory = 0;

    public void store(int value) {
        this.memory = value;
    }

    public int recall() {
        return memory;
    }

    // Static method
    public static int multiply(int a, int b) {
        return a * b;
    }

    public static void main(String[] args) {
        // Static method invocation via class name
        int product = Calculator.multiply(6, 7);
        System.out.println("Product: " + product);

        // Instance method invocation via object reference
        Calculator calc = new Calculator();
        calc.store(42);
        System.out.println("Memory: " + calc.recall());
    }
}
```

**Expected Output:**

```
Product: 42
Memory: 42
```

**Why This Output Occurs:**
- `Calculator.multiply(6, 7)` invokes the static method directly on the class, returning `42`.
- `calc.store(42)` invokes the instance method on the `calc` object, storing `42` in its `memory` field.
- `calc.recall()` retrieves the stored value.

### Real-World Cases

- **Event handling**: `button.setOnClickListener(handler)` invokes a method when clicked.
- **Service orchestration**: `orderService.process(order)` delegates to the service layer.
- **Utility usage**: `Math.sqrt(16)` invokes a static utility method.

### References

- CMU – Methods Handout (Flow of Control) - http://www.cs.cmu.edu/afs/cs.cmu.edu/user/mrmiller/www/15-110/Handouts/methods-4.pdf
- Dev.java – Calling Methods and Constructors - https://dev.java/learn/classes-objects/calling-methods-constructors/
- Java Language Specification – Method Invocation Expressions - https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.12


## Core Concept 3: Parameters and Arguments

### Definitions

**Core Definition**: **Parameters** are the variables declared in a method's signature that receive values when the method is invoked. **Arguments** are the actual values passed to the method during invocation.

**Technical Definition**: The declaration for a method or constructor declares the number and type of the arguments for that method. **Parameters** refer to the list of variables in a method declaration. **Arguments** are the actual values that are passed in when the method is invoked. When a method is invoked, the arguments used must match the declaration's parameters in type and order. Java supports **varargs** (variable-length argument lists) using the ellipsis (`...`) syntax, allowing an arbitrary number of arguments of a given type.

**Beginner-Friendly Explanation**: Parameters are the empty slots in a recipe that say "add X cups of flour." Arguments are the actual ingredients you put in—"2 cups of flour." You must put the right type and amount, or the recipe won't work.

### Purposes

- To pass data into a method for processing.
- To make methods flexible and reusable across different inputs.
- To establish a clear contract for what the method expects.
- To support variable numbers of inputs through varargs.

### Syntax Rules and Structure

**Complete General Syntax (Parameter List):**

```java
(Type1 paramName1, Type2 paramName2, ..., TypeN paramNameN)
```

**Complete General Syntax (Varargs):**

```java
(Type... paramName)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Type` | The data type of the parameter (primitive or reference). |
| `paramName` | The name used within the method body to refer to the argument. |
| `...` | Ellipsis for varargs (must be the last parameter). |

**Syntax Rules:**

- Parameter names must be unique within the method's scope; they cannot duplicate each other or local variables.
- A parameter can shadow a class field; use `this.fieldName` to access the field.
- Arguments must match parameters in type, order, and number.
- Varargs allows zero or more arguments; inside the method, the parameter is treated as an array.
- Primitive arguments are passed by value; reference arguments copy the reference (but the object's fields can be modified).

**Constraints and Limitations:**

- Parameter names should be descriptive; single letters like `x` should be avoided except in simple cases.
- Varargs must be the last parameter in the list.
- You cannot have two parameters with the same name in the same method.
- Shadowing fields can make code confusing; use `this` for clarity.

### Annotated Complete Code Examples

**Example 1: Parameters, Arguments, and Varargs**

```java
public class ParameterDemo {

    // Method with two parameters
    public static int add(int a, int b) {
        return a + b;
    }

    // Method with varargs
    public static int sum(int... numbers) {
        int total = 0;
        for (int n : numbers) { // numbers is treated as an array
            total += n;
        }
        return total;
    }

    // Method where parameter shadows field
    private String name;

    public void setName(String name) {
        this.name = name; // 'this.name' refers to the field
    }

    public String getName() {
        return name;
    }

    public static void main(String[] args) {
        // Arguments: 3 and 4
        System.out.println("add(3, 4) = " + add(3, 4));

        // Varargs: zero, three, and five arguments
        System.out.println("sum() = " + sum());
        System.out.println("sum(1, 2, 3) = " + sum(1, 2, 3));
        System.out.println("sum(1, 2, 3, 4, 5) = " + sum(1, 2, 3, 4, 5));

        // Parameter shadowing
        ParameterDemo demo = new ParameterDemo();
        demo.setName("Java");
        System.out.println("Name: " + demo.getName());
    }
}
```

**Expected Output:**

```
add(3, 4) = 7
sum() = 0
sum(1, 2, 3) = 6
sum(1, 2, 3, 4, 5) = 15
Name: Java
```

**Why This Output Occurs:**
- `add(3, 4)` passes arguments `3` and `4` to parameters `a` and `b`, returning `7`.
- `sum()` with no arguments returns `0` because the varargs array is empty.
- `sum(1, 2, 3)` passes three arguments, summing to `6`.
- `sum(1, 2, 3, 4, 5)` passes five arguments, summing to `15`.
- `setName("Java")` shadows the field `name`; `this.name = name` assigns the parameter to the field.

### Real-World Cases

- **Logging**: `logger.info("User {} logged in", userId)` uses varargs.
- **Formatting**: `String.format("%s is %d years old", name, age)`.
- **Mathematical operations**: `Math.max(a, b)`, `Math.pow(base, exponent)`.
- **Builder methods**: `new StringBuilder().append("a").append("b")`.

### References

- Oracle Java Tutorials – Passing Information to a Method or Constructor - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Dev.java – Passing Information to a Method or a Constructor - https://dev.java/learn/classes-objects/calling-methods-constructors/
- Java Language Specification – Formal Parameters - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.4.1


## Core Concept 4: Return Values

### Definitions

**Core Definition**: A return value is the result that a method sends back to the code that invoked it, specified by the method's return type and the `return` statement.

**Technical Definition**: If a method is to return a value, its return type must be a primitive data type or a reference type (not `void`). The `return` statement inside the method specifies the value to be returned. When the `return` statement executes, the method terminates immediately, and the value is passed back to the caller. The return type declares what kind of data the method returns.

**Beginner-Friendly Explanation**: A return value is like the finished dish from a recipe. The recipe says "this makes a cake" (return type), and when you're done, you hand over the cake to whoever asked for it.

### Purposes

- To provide the result of a computation to the calling code.
- To enable method chaining and expression composition.
- To communicate success or failure through returned values.
- To allow methods to be used as part of larger expressions.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
ReturnType methodName(parameters) {
    // ...
    return value; // value must be assignable to ReturnType
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ReturnType` | The type of the returned value (primitive or reference). |
| `return` | Keyword that exits the method and provides the value. |
| `value` | An expression matching the return type. |

**Syntax Rules:**

- The `return` statement must appear on all code paths for non-`void` methods.
- The returned value must be assignable to the declared return type (or a subtype).
- A method can return at most one value (use objects/arrays for multiple values).
- The `return` keyword can also be used in `void` methods to exit early (without a value).

**Constraints and Limitations:**

- Return type is not part of the method signature; you cannot overload based on return type alone.
- Returning `null` from a reference-returning method can cause `NullPointerException` if the caller doesn't check.
- Returning large objects can have performance implications.

### Annotated Complete Code Examples

**Example 1: Return Values in Action**

```java
public class ReturnValueDemo {

    // Returns an int
    public static int square(int n) {
        return n * n;
    }

    // Returns a String
    public static String repeat(String s, int times) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < times; i++) {
            sb.append(s);
        }
        return sb.toString();
    }

    // Returns a boolean
    public static boolean isEven(int n) {
        return n % 2 == 0;
    }

    // Early return in a void method
    public static void printPositive(int n) {
        if (n <= 0) {
            return; // Exit early
        }
        System.out.println("Positive: " + n);
    }

    public static void main(String[] args) {
        System.out.println("square(5) = " + square(5));
        System.out.println("repeat(\"ab\", 3) = " + repeat("ab", 3));
        System.out.println("isEven(4) = " + isEven(4));
        System.out.println("isEven(7) = " + isEven(7));
        printPositive(10);
        printPositive(-5); // No output
    }
}
```

**Expected Output:**

```
square(5) = 25
repeat("ab", 3) = ababab
isEven(4) = true
isEven(7) = false
Positive: 10
```

**Why This Output Occurs:**
- `square(5)` returns `25`, which is concatenated with the string.
- `repeat("ab", 3)` returns `"ababab"`.
- `isEven(4)` returns `true`; `isEven(7)` returns `false`.
- `printPositive(10)` prints the message; `printPositive(-5)` returns early without printing.

### Real-World Cases

- **Getters**: `getName()`, `getBalance()`, `getSize()`.
- **Computation**: `calculateTotal()`, `computeAverage()`, `findMax()`.
- **Validation**: `isValid()`, `hasPermission()`, `isEmpty()`.
- **Factory methods**: `createUser()`, `buildOrder()`, `of()`.

### References

- CMU – Methods Handout (Return Type) - http://www.cs.cmu.edu/afs/cs.cmu.edu/user/mrmiller/www/15-110/Handouts/methods-4.pdf
- MIT – void Keyword - https://people.csail.mit.edu/phw/OnToJava/ONTOJAVA111.HTML
- Republic of the Philippines – Programming II Java Methods - http://phlconnect.ched.gov.ph/admin/uploads/fa7cdfad1a5aaf8370ebeda47a1ff1c3/Programming-II-Java-Methods.pdf


## Core Concept 5: `void`

### Definitions

**Core Definition**: `void` is a keyword used as a method's return type to indicate that the method does not return a value.

**Technical Definition**: When Java sees `void` used as though it were a return-value data type, Java knows that nothing is to be returned. A `void` method performs an action (such as printing or modifying state) but does not provide a result to the caller. The `return` statement in a `void` method can only be used to exit early, without a value.

**Beginner-Friendly Explanation**: A `void` method is like a recipe that says "preheat the oven." You do it, but you don't get a dish back. You might just do it for the sake of doing it.

### Purposes

- To declare methods that perform actions without producing a result.
- To perform side effects such as printing, logging, or modifying object state.
- To signal that the method's purpose is an operation, not a computation.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public void methodName(parameters) {
    // method body
    // no return value
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `void` | The return type indicating no value is returned. |
| `methodName` | The method identifier. |
| `(parameters)` | The parameter list. |
| `{ ... }` | The method body. |

**Syntax Rules:**

- A `void` method cannot be used in an expression that expects a value.
- The `return` statement in a `void` method can only be used without a value (e.g., `return;`).
- A `void` method can still have side effects (modify fields, print output, call other methods).
- `void` is not a type; it cannot be used for variables or fields.

**Constraints and Limitations:**

- Cannot assign the result of a `void` method to a variable.
- Cannot use `void` methods in `return` statements of other methods.
- `void` is not a class; `Void` (capital V) is the corresponding wrapper class for reflective purposes.

### Annotated Complete Code Examples

**Example 1: void Methods vs. Value-Returning Methods**

```java
public class VoidDemo {
    private int counter = 0;

    // void method: performs an action, no return value
    public void increment() {
        counter++;
    }

    // void method: prints a message
    public void greet(String name) {
        System.out.println("Hello, " + name + "!");
    }

    // value-returning method: returns the counter
    public int getCounter() {
        return counter;
    }

    public static void main(String[] args) {
        VoidDemo demo = new VoidDemo();

        // Invoking void methods as statements
        demo.greet("Alice");
        demo.increment();
        demo.increment();

        // Using the return value of a non-void method
        int count = demo.getCounter();
        System.out.println("Counter: " + count);
    }
}
```

**Expected Output:**

```
Hello, Alice!
Counter: 2
```

**Why This Output Occurs:**
- `greet("Alice")` is a `void` method that prints a message and returns nothing.
- `increment()` is a `void` method that modifies the `counter` field.
- `getCounter()` returns the `int` value of `counter`, which is `2` after two increments.
- The `void` methods are invoked as statements, while `getCounter()` is used in an expression.

### Real-World Cases

- **Setters**: `setName(String name)`, `setBalance(double balance)`.
- **Printing/Logging**: `System.out.println()`, `logger.info()`.
- **Event handlers**: `onClick()`, `onSubmit()`.
- **State modifiers**: `start()`, `stop()`, `reset()`.

### References

- MIT – void Keyword - https://people.csail.mit.edu/phw/OnToJava/ONTOJAVA111.HTML
- CMU – Methods Handout (void) - http://www.cs.cmu.edu/afs/cs.cmu.edu/user/mrmiller/www/15-110/Handouts/methods-4.pdf
- Republic of the Philippines – Programming II Java Methods - http://phlconnect.ched.gov.ph/admin/uploads/fa7cdfad1a5aaf8370ebeda47a1ff1c3/Programming-II-Java-Methods.pdf


## Core Concept 6: Method Signatures (Name and Parameter Types)

### Definitions

**Core Definition**: A method signature is the combination of a method's name and its parameter types, which uniquely identifies the method within its class.

**Technical Definition**: Two of the components of a method declaration comprise the method signature—the method's name and the parameter types. The return type and modifiers are not part of the signature. This means methods can share the same name as long as their parameter lists differ (overloading).

**Beginner-Friendly Explanation**: A method signature is like a person's full name. Two people can have the same first name (method name) but different last names (parameter types), and they're distinct individuals.

### Purposes

- To uniquely identify methods within a class.
- To enable method overloading (same name, different parameters).
- To allow the compiler to resolve method calls based on argument types.
- To serve as a key for method resolution in inheritance and interfaces.

### Syntax Rules and Structure

**Complete General Syntax (Signature):**

```java
methodName(ParameterType1, ParameterType2, ..., ParameterTypeN)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methodName` | The identifier of the method. |
| `(ParameterType1, ...)` | The ordered list of parameter types (not names). |

**Examples of Signatures:**

| Declaration | Signature |
|-------------|-----------|
| `public void draw(String s)` | `draw(String)` |
| `public void draw(int i)` | `draw(int)` |
| `public void draw(double f)` | `draw(double)` |
| `public void draw(int i, double f)` | `draw(int, double)` |

**Syntax Rules:**

- The return type is NOT part of the signature.
- The `static` modifier is NOT part of the signature.
- Parameter names are NOT part of the signature; only types matter.
- Two methods with the same name and same parameter types are duplicates, even if return types differ.
- Overloading is based on the signature: same name, different parameter lists.

**Constraints and Limitations:**

- You cannot overload by return type alone.
- You cannot overload by parameter names alone.
- Varargs methods have a signature based on the array type (e.g., `sum(int[])`).
- Type erasure affects generic methods: `<T> void method(T t)` and `void method(Object o)` have the same erasure signature.

### Annotated Complete Code Examples

**Example 1: Method Overloading by Signature**

```java
public class OverloadDemo {

    // Signature: draw(String)
    public void draw(String s) {
        System.out.println("Drawing string: " + s);
    }

    // Signature: draw(int)
    public void draw(int i) {
        System.out.println("Drawing int: " + i);
    }

    // Signature: draw(double)
    public void draw(double f) {
        System.out.println("Drawing double: " + f);
    }

    // Signature: draw(int, double)
    public void draw(int i, double f) {
        System.out.println("Drawing int and double: " + i + ", " + f);
    }

    public static void main(String[] args) {
        OverloadDemo demo = new OverloadDemo();

        // The compiler resolves which method to call based on argument types
        demo.draw("Hello");      // Calls draw(String)
        demo.draw(42);           // Calls draw(int)
        demo.draw(3.14);         // Calls draw(double)
        demo.draw(10, 2.71);     // Calls draw(int, double)
    }
}
```

**Expected Output:**

```
Drawing string: Hello
Drawing int: 42
Drawing double: 3.14
Drawing int and double: 10, 2.71
```

**Why This Output Occurs:**
- Each `draw` method has a distinct signature: `draw(String)`, `draw(int)`, `draw(double)`, `draw(int, double)`.
- The compiler selects the appropriate method based on the type of the argument passed.
- `draw("Hello")` matches `draw(String)`; `draw(42)` matches `draw(int)`; etc.

### Real-World Cases

- **Convenience methods**: `println()` overloads for different types.
- **Constructors**: `ArrayList()` vs. `ArrayList(int initialCapacity)`.
- **API design**: `valueOf(int)`, `valueOf(String)`, `valueOf(double)`.
- **Math utilities**: `abs(int)`, `abs(long)`, `abs(double)`, `abs(float)`.

### References

- Oracle Java Tutorials – Defining Methods (Overloading) - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html
- Java Language Specification – Method Signature - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.4.2
- CMU – Methods Handout - http://www.cs.cmu.edu/afs/cs.cmu.edu/user/mrmiller/www/15-110/Handouts/methods-4.pdf


## Core Concept 7: Method Scope

### Definitions

**Core Definition**: Method scope refers to the region of a program where a method's name is visible and can be invoked, as well as the region where its parameters and local variables exist.

**Technical Definition**: A method is a member of a class and is visible within the class body and, depending on access modifiers, from other classes. Parameters and local variables declared inside a method are scoped to that method; they cannot be accessed from outside. A method's type variables (generics) are scoped to the method declaration and body. Method declarations cannot be nested inside other methods; they must be direct members of a class.

**Beginner-Friendly Explanation**: Method scope is like the visibility of a recipe in a cookbook. A recipe is available to anyone who has the cookbook (class scope). The ingredients you use while cooking (local variables) are only available during that cooking session—they don't exist before or after.

### Purposes

- To control where a method can be invoked from.
- To ensure that local variables and parameters do not leak outside the method.
- To prevent naming conflicts between different methods' variables.
- To support encapsulation and information hiding.

### Syntax Rules and Structure

**Scope Rules:**

| Entity | Scope |
|--------|-------|
| Method name | Visible within the class (and outside based on access modifier). |
| Parameters | Visible only within the method body. |
| Local variables | Visible from declaration to the end of the block. |
| Type parameters | Visible within the method declaration and body. |

**Syntax Rules:**

- A method cannot be declared inside another method.
- Parameters and local variables are destroyed when the method returns.
- A local variable cannot have the same name as a parameter in the same method.
- A parameter can shadow a field; use `this.field` to access the field.
- Method declarations can appear in any order within a class; forward references are allowed.

**Constraints and Limitations:**

- Local variables must be initialized before use; they have no default values.
- Parameters are initialized with the arguments provided at invocation.
- The scope of a local variable begins at its declaration and ends at the closing brace of the block.
- Variables declared in a `for` loop header are scoped to the loop body.

### Annotated Complete Code Examples

**Example 1: Method and Variable Scope**

```java
public class ScopeDemo {
    // Field: visible throughout the class
    private int fieldValue = 100;

    // Method with a parameter and local variable
    public void demonstrateScope(int parameter) {
        // parameter is visible here
        System.out.println("Parameter: " + parameter);

        // Local variable: visible only in this method
        int localVar = parameter * 2;
        System.out.println("Local variable: " + localVar);

        // Field is visible and accessible
        System.out.println("Field: " + fieldValue);

        // Block scope
        {
            int blockVar = 10;
            System.out.println("Block variable: " + blockVar);
        }
        // blockVar is out of scope here

        // Loop scope
        for (int i = 0; i < 3; i++) {
            System.out.println("Loop variable i: " + i);
        }
        // i is out of scope here
    }

    public static void main(String[] args) {
        ScopeDemo demo = new ScopeDemo();
        demo.demonstrateScope(5);
    }
}
```

**Expected Output:**

```
Parameter: 5
Local variable: 10
Field: 100
Block variable: 10
Loop variable i: 0
Loop variable i: 1
Loop variable i: 2
```

**Why This Output Occurs:**
- `parameter` is visible throughout the `demonstrateScope` method.
- `localVar` is declared inside the method and is visible until the method ends.
- `fieldValue` is a class field and is accessible from the method.
- `blockVar` is declared inside a block and is only visible within that block.
- `i` is declared in the `for` loop header and is only visible within the loop.

### Real-World Cases

- **Helper methods**: Private methods visible only within the class.
- **Temporary variables**: Local variables used for intermediate calculations.
- **Loop counters**: Variables scoped to loops to avoid naming conflicts.
- **Parameter shadowing**: Constructor parameters with the same name as fields, resolved with `this`.

### References

- Oracle Java Tutorials – Defining Methods - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html
- CMU – Methods Handout (Scope) - http://www.cs.cmu.edu/afs/cs.cmu.edu/user/mrmiller/www/15-110/Handouts/methods-4.pdf
- Cornell – Transitioning to Java (Local Variables) - https://www.cs.cornell.edu/courses/cs2110/2026sp/resources/transition/
- OpenJDK – MethodScope (Generics) - https://cr.openjdk.org/~avstepan/8133651/jdk.00/raw_files/old/src/java.base/share/classes/sun/reflect/generics/scope/MethodScope.java


## Core Concept 8: Access Modifiers (public, private, protected, package-private) and Visibility

### Definitions

**Core Definition**: Access modifiers are keywords that control the visibility and accessibility of classes, methods, and fields from other parts of a program.

**Technical Definition**: Access level modifiers determine whether other classes can use a particular field or invoke a particular method. There are four access levels for members: `public`, `protected`, package-private (no modifier), and `private`. The access rules are: `public` is accessible from anywhere; `protected` is accessible within the package and by subclasses in other packages; package-private is accessible only within the same package; `private` is accessible only within the same class.

**Beginner-Friendly Explanation**: Access modifiers are like the locks on different rooms in a house. `public` is the front yard—anyone can enter. `protected` is the living room—family (subclasses) and neighbors (same package) can enter. Package-private is the kitchen—only people in the house (same package) can enter. `private` is your bedroom—only you (the class) can enter.

### Purposes

- To enforce encapsulation by hiding implementation details.
- To control what parts of a class are part of its public API.
- To minimize the accessibility of members for security and maintainability.
- To prevent unintended modification or misuse of internal state.

### Syntax Rules and Structure

**Access Modifier Table:**

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `public` | Y | Y | Y | Y |
| `protected` | Y | Y | Y | N |
| (none) package-private | Y | Y | N | N |
| `private` | Y | N | N | N |

*Source: Oracle Java Tutorials, SEI CERT*

**Complete General Syntax:**

```java
[access_modifier] ReturnType methodName(parameters) { ... }
```

**Component Breakdown:**

| Modifier | Description |
|----------|-------------|
| `public` | Accessible from any class everywhere. |
| `protected` | Accessible within package and by subclasses in other packages. |
| (none) | Package-private: accessible only within the same package. |
| `private` | Accessible only within the same class. |

**Syntax Rules:**

- A class always has access to its own members, regardless of access level.
- `private` members are not inherited by subclasses (though they exist in the object).
- `protected` members are accessible to subclasses even in different packages.
- Package-private members are accessible by any class in the same package (with the same class loader).
- Use the most restrictive access level that makes sense.

**Constraints and Limitations:**

- Top-level classes can only be `public` or package-private.
- `private` and `protected` cannot be used for top-level classes.
- Reducing accessibility can break existing code that depends on public members.
- `protected` is often overused; prefer `private` with protected accessors if needed.

### Annotated Complete Code Examples

**Example 1: Access Modifiers in Action**

```java
// File: AccessDemo.java
package com.example.access;

public class AccessDemo {
    public int publicField = 1;
    protected int protectedField = 2;
    int packagePrivateField = 3;  // no modifier
    private int privateField = 4;

    public void publicMethod() {
        System.out.println("Public method");
    }

    protected void protectedMethod() {
        System.out.println("Protected method");
    }

    void packagePrivateMethod() {
        System.out.println("Package-private method");
    }

    private void privateMethod() {
        System.out.println("Private method");
    }

    // Public method can access all private members
    public void accessAll() {
        System.out.println(publicField);
        System.out.println(protectedField);
        System.out.println(packagePrivateField);
        System.out.println(privateField);
        privateMethod(); // Private method is accessible within the class
    }

    public static void main(String[] args) {
        AccessDemo demo = new AccessDemo();
        demo.accessAll();
    }
}
```

**Expected Output:**

```
1
2
3
4
Private method
```

**Why This Output Occurs:**
- The `accessAll` method is in the same class, so it can access all members regardless of access modifier.
- `privateMethod()` is called from within the class, which is allowed.
- The fields are printed in order: `publicField`, `protectedField`, `packagePrivateField`, `privateField`.

**Example 2: Visibility Across Classes**

```java
// File: SamePackageClass.java
package com.example.access;

public class SamePackageClass {
    public void testAccess() {
        AccessDemo demo = new AccessDemo();
        System.out.println(demo.publicField);           // OK
        System.out.println(demo.protectedField);        // OK (same package)
        System.out.println(demo.packagePrivateField);   // OK (same package)
        // System.out.println(demo.privateField);       // Compile error: private
    }
}
```

**Expected Output (from SamePackageClass):**

```
1
2
3
```

**Why This Output Occurs:**
- `SamePackageClass` is in the same package as `AccessDemo`, so it can access public, protected, and package-private members.
- `privateField` is not accessible from a different class, even in the same package.

### Real-World Cases

- **API classes**: Public methods and classes form the exposed API.
- **Internal helpers**: Private methods hide implementation details.
- **Inheritance**: Protected methods allow subclasses to extend behavior.
- **Package organization**: Package-private members are shared within a package but hidden from the world.

### References

- Oracle Java Tutorials – Controlling Access to Members of a Class - https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html
- SEI CERT – OBJ51-J: Minimize the accessibility of classes and their members - https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/recommendations/object-orientation-obj/obj51-j/
- SEI CERT – SEC01-J: Minimize accessibility of classes and their members - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88492209


## Core Concept 9: Static vs. Instance Methods (the `static` keyword)

### Definitions

**Core Definition**: **Instance methods** are methods that operate on an object's state and require an instance to be invoked. **Static methods** are methods that belong to the class itself, do not depend on instance state, and can be invoked without creating an object.

**Technical Definition**: Instance methods depend on instance state and perform actions "on or for" an object. Static methods do NOT depend on instance state and perform actions common to the entire class; they are declared with the `static` keyword. Static methods cannot access instance fields or instance methods directly. Instance methods can access both instance and static members.

**Beginner-Friendly Explanation**: Instance methods are like personal skills—you need a person (object) to use them. "Alice can swim" requires Alice. Static methods are like universal truths—"water freezes at 0°C" doesn't depend on any particular person. You can state it without needing a specific object.

### Purposes

- **Instance methods**: To perform operations that depend on or modify object state.
- **Static methods**: To provide utility functions, factory methods, or operations that don't require object state.
- To clearly distinguish between behavior that is per-object vs. behavior that is common to the class.
- To support design patterns like Singleton (private constructor + static factory).

### Syntax Rules and Structure

**Complete General Syntax (Instance Method):**

```java
public ReturnType methodName(parameters) {
    // can access this.field and other instance methods
}
// Invocation: object.methodName(args)
```

**Complete General Syntax (Static Method):**

```java
public static ReturnType methodName(parameters) {
    // cannot access this.field or instance methods directly
}
// Invocation: ClassName.methodName(args)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `static` | Keyword indicating a class-level method. |
| Instance method | No `static`; requires an object for invocation. |

**Comparison Table:**

| Aspect | Instance Method | Static Method |
|--------|-----------------|---------------|
| Requires object | Yes | No |
| Accesses instance fields | Yes | No (directly) |
| Accesses static fields | Yes | Yes |
| Can be overridden | Yes | No (hidden, not overridden) |
| Invocation | `object.method()` | `ClassName.method()` |

**Syntax Rules:**

- Static methods can be invoked via the class name (preferred) or an instance reference (discouraged).
- Instance methods can call static methods directly.
- Static methods cannot use `this` or `super`.
- Static methods cannot be `abstract` (in classes) or overridden (they are hidden).
- The `main` method is static so it can be invoked without creating an instance.

**Constraints and Limitations:**

- Static methods cannot access instance state, making them unsuitable for operations that depend on object-specific data.
- Making a method static can break polymorphism and make testing harder.
- Public static methods tightly couple callers to the implementation.
- If a method might later need instance state, making it static now creates refactoring overhead.

### Annotated Complete Code Examples

**Example 1: Instance vs. Static Methods**

```java
public class Counter {
    // Instance field: each object has its own count
    private int instanceCount = 0;

    // Static field: shared by all objects
    private static int totalCount = 0;

    // Instance method: operates on instance state
    public void incrementInstance() {
        instanceCount++;
        totalCount++; // Can access static field
    }

    // Instance method: reads instance state
    public int getInstanceCount() {
        return instanceCount;
    }

    // Static method: operates on class state only
    public static int getTotalCount() {
        return totalCount;
        // Cannot access instanceCount directly
    }

    // Static utility method: no state dependency
    public static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        // Static method invocation via class name
        System.out.println("Static add: " + Counter.add(3, 4));
        System.out.println("Total count (before): " + Counter.getTotalCount());

        // Create two objects
        Counter c1 = new Counter();
        Counter c2 = new Counter();

        // Instance method invocation
        c1.incrementInstance();
        c1.incrementInstance();
        c2.incrementInstance();

        // Instance state is per-object
        System.out.println("c1 instance count: " + c1.getInstanceCount());
        System.out.println("c2 instance count: " + c2.getInstanceCount());

        // Static state is shared
        System.out.println("Total count (after): " + Counter.getTotalCount());
    }
}
```

**Expected Output:**

```
Static add: 7
Total count (before): 0
c1 instance count: 2
c2 instance count: 1
Total count (after): 3
```

**Why This Output Occurs:**
- `Counter.add(3, 4)` is static and returns `7` without any object.
- `totalCount` starts at `0` (no objects created).
- `c1.incrementInstance()` is called twice, so `c1`'s `instanceCount` is `2`.
- `c2.incrementInstance()` is called once, so `c2`'s `instanceCount` is `1`.
- `totalCount` is shared: `2 + 1 = 3`.

### Real-World Cases

- **Utility classes**: `Math.sqrt()`, `Collections.sort()`, `Arrays.asList()` are static.
- **Factory methods**: `LocalDate.now()`, `List.of()`, `Optional.empty()` are static.
- **Singleton pattern**: `getInstance()` is static and returns the single instance.
- **Domain models**: `BankAccount.deposit()` is an instance method because it depends on the account's balance.

### References

- Cornell – Class vs. Instance Methods - https://www.cs.cornell.edu/courses/cs202/1999FA/lecture/f99-cs202-l1/tsld015.htm
- Stack Overflow – Decision of Static methods vs instance methods in java - https://stackoverflow.com/questions/7208444/decision-of-static-methods-vs-instance-methods-in-java
- Oracle Java Tutorials – Defining Methods - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html


## References

- Oracle Java Tutorials – Defining Methods - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html
- Oracle Java Tutorials – Passing Information to a Method or a Constructor - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Oracle Java Tutorials – Controlling Access to Members of a Class - https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html
- Java Language Specification – Method Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.4
- Java Language Specification – Method Invocation Expressions - https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html#jls-15.12
- Dev.java – Calling Methods and Constructors - https://dev.java/learn/classes-objects/calling-methods-constructors/
- CMU – Methods Handout - http://www.cs.cmu.edu/afs/cs.cmu.edu/user/mrmiller/www/15-110/Handouts/methods-4.pdf
- MIT – void Keyword - https://people.csail.mit.edu/phw/OnToJava/ONTOJAVA111.HTML
- Cornell – Class vs. Instance Methods - https://www.cs.cornell.edu/courses/cs202/1999FA/lecture/f99-cs202-l1/tsld015.htm
- Cornell – Transitioning to Java - https://www.cs.cornell.edu/courses/cs2110/2026sp/resources/transition/
- SEI CERT – OBJ51-J: Minimize the accessibility of classes and their members - https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/recommendations/object-orientation-obj/obj51-j/
- SEI CERT – SEC01-J: Minimize accessibility of classes and their members - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88492209
- OpenJDK – MethodScope (Generics) - https://cr.openjdk.org/~avstepan/8133651/jdk.00/raw_files/old/src/java.base/share/classes/sun/reflect/generics/scope/MethodScope.java
- Republic of the Philippines – Programming II Java Methods - http://phlconnect.ched.gov.ph/admin/uploads/fa7cdfad1a5aaf8370ebeda47a1ff1c3/Programming-II-Java-Methods.pdf