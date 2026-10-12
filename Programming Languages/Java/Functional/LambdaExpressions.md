# Java Lambda Expressions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A lambda expression is an anonymous function — a block of code that can be passed around and executed later — that provides a concise way to represent an instance of a functional interface (an interface with exactly one abstract method).

**Technical Definition:** "A lambda expression is like a method: it provides a list of formal parameters and a body — an expression or block — expressed in terms of those parameters. Lambda expressions are compatible in an assignment context, invocation context, or casting context with a target type T if T is a functional interface type and the expression is congruent with the function type of the ground target type derived from T". "Lambda expressions let you express instances of single-method classes more compactly".

**Beginner-Friendly Explanation:** Think of a lambda expression as a mini-method that you can write inline, without giving it a name, and hand it to another method as if it were a value. Instead of writing a whole class just to define one tiny behaviour, you write a short expression that says "given these inputs, do this."

### Key Characteristics

- **Conciseness:** Lambda expressions eliminate the boilerplate of anonymous classes.
- **Anonymous:** Lambdas have no name; they are defined inline where they are used.
- **Functional:** They implement a functional interface — an interface with exactly one abstract method.
- **Lexically Scoped:** Lambda expressions do not introduce a new scope; `this` refers to the enclosing class instance.
- **Target Typed:** The compiler infers the parameter types and the functional interface from the context.
- **Capturing:** Lambdas can capture local variables from their enclosing scope if those variables are final or effectively final.

### Prerequisites

- Basic understanding of Java interfaces and abstract methods
- Familiarity with anonymous inner classes
- Knowledge of generics (type parameters)
- Understanding of the `java.util.function` package

### Related Programming Areas

- **Functional Programming:** Lambdas bring functional-style programming to Java.
- **Stream API:** The Streams API relies heavily on lambda expressions for filtering, mapping, and reducing.
- **Event Handling:** GUI programming uses lambdas for event listeners.
- **Concurrency:** `CompletableFuture` and parallel streams use lambdas.
- **Method References:** A shorthand notation for lambdas that call existing methods.

### Core Concepts / Features

---

## 1. Lambda Expressions

### Definitions

**Core Definition:** A lambda expression is an anonymous function that can be treated as a value and passed to methods or stored in variables.

**Technical Definition:** "A lambda expression is compatible in an assignment context, invocation context, or casting context with a target type T if T is a functional interface type". The JLS defines lambda expressions as expressions that "encapsulate a single unit of behavior and pass it to other code".

**Beginner-Friendly Explanation:** A lambda expression is like a sticky note with instructions written on it. You can hand that sticky note to someone (a method) and say "here's what I want you to do." The sticky note doesn't have a name; it just contains the instructions.

### Purposes

- To treat functionality as a method argument, or code as data
- To express instances of single-method interfaces more compactly
- To enable functional programming patterns in Java
- To reduce boilerplate code associated with anonymous classes
- To improve readability for simple behavioural implementations

### Syntax Rules and Structure

**Complete General Syntax:**

```
(parameters) -> expression
(parameters) -> { statements; }
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `parameters` | A comma-separated list of formal parameters, enclosed in parentheses. Parentheses may be omitted for a single parameter with an inferred type. |
| `->` | The arrow token (lambda operator) separating parameters from the body. |
| `expression` | A single expression whose value is returned. |
| `{ statements; }` | A block body enclosed in braces; requires a `return` statement for non-void functional interfaces. |

**Syntax Rules:**

- If the body is a single expression, braces and `return` are optional.
- If the body is a block, braces are required, and a `return` statement is needed for non-void functional interfaces.
- Parameter types may be omitted if they can be inferred from the target type.
- Parentheses around a single inferred-type parameter may be omitted.
- A lambda expression does not introduce a new level of scoping.

**Constraints and Limitations:**

- Lambdas are limited to functional interfaces; they cannot be used to create instances of abstract classes.
- Lambdas cannot declare fields or additional methods.
- Lambdas cannot reference themselves (no recursion by name).
- Lambdas share with anonymous classes the property that they cannot be reliably serialized across implementations.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Lambda Expression**

```java
// File: LambdaBasic.java
public class LambdaBasic {
    public static void main(String[] args) {
        // Step 1: Define a functional interface
        // A functional interface has exactly one abstract method
        interface Greeting {
            void sayHello(String name);
        }

        // Step 2: Implement the interface using a lambda expression
        // The lambda takes one parameter 'name' and prints a message
        Greeting greeting = (name) -> System.out.println("Hello, " + name + "!");

        // Step 3: Invoke the lambda through the interface reference
        greeting.sayHello("Alice");        // Output: Hello, Alice!
        greeting.sayHello("Bob");          // Output: Hello, Bob!

        // Step 4: Another lambda with a block body
        Greeting formalGreeting = (name) -> {
            String title = "Dear";
            System.out.println(title + " " + name + ", welcome.");
        };

        formalGreeting.sayHello("Dr. Smith");     // Dear Dr. Smith, welcome.
    }
}
```

**Expected Output:**
```
Hello, Alice!
Hello, Bob!
Dear Dr. Smith, welcome.
```

**Step-by-Step Explanation:**

1. A functional interface `Greeting` is declared with a single abstract method `sayHello(String)`.
2. The lambda `(name) -> System.out.println("Hello, " + name + "!")` implements this interface. The parameter `name` is inferred as `String` from the target type.
3. When `greeting.sayHello("Alice")` is called, the lambda body executes, printing `"Hello, Alice!"`.
4. The second lambda uses a block body with a local variable `title`, demonstrating that lambdas can contain multiple statements.

**Example 2: Lambda with Return Value**

```java
// File: LambdaReturn.java
public class LambdaReturn {
    public static void main(String[] args) {
        // Functional interface with a return value
        interface Calculator {
            int operate(int a, int b);
        }

        // Lambda with an expression body (return is implicit)
        Calculator add = (a, b) -> a + b;
        Calculator subtract = (a, b) -> a - b;

        // Lambda with a block body (return is explicit)
        Calculator multiply = (a, b) -> {
            int result = a * b;
            return result;
        };

        System.out.println("Add: " + add.operate(10, 5));        // Output: Add: 15
        System.out.println("Subtract: " + subtract.operate(10, 5)); // Output: Subtract: 5
        System.out.println("Multiply: " + multiply.operate(10, 5)); // Output: Multiply: 50
    }
}
```

**Expected Output:**
```
Add: 15
Subtract: 5
Multiply: 50
```

**Step-by-Step Explanation:**

1. The `Calculator` interface declares `int operate(int a, int b)`.
2. The `add` lambda uses an expression body: `a + b` is evaluated and returned implicitly.
3. The `multiply` lambda uses a block body: a local variable `result` is computed, and `return` explicitly sends it back.
4. The compiler infers the parameter types `int` and `int` from the target type `Calculator`.

### Real-World Cases

- **Sorting:** `Collections.sort(list, (a, b) -> a.compareTo(b))` sorts a list using a lambda comparator.
- **Event Handling:** `button.addActionListener(e -> System.out.println("Clicked!"))` handles button clicks.
- **Stream Filtering:** `list.stream().filter(x -> x > 10)` filters a stream using a lambda predicate.
- **Thread Creation:** `new Thread(() -> System.out.println("Running"))` creates a thread with a lambda `Runnable`.

---

## 2. Lambda Syntax

### Definitions

**Core Definition:** Lambda syntax refers to the grammatical rules that define how lambda expressions are written in Java.

**Technical Definition:** "The syntax of lambda expressions includes rules for the new kinds of parameter lists and statement lists". A lambda expression consists of a parameter list, an arrow token (`->`), and a body (either an expression or a block).

**Beginner-Friendly Explanation:** Lambda syntax is the "grammar" for writing lambdas. It tells you where to put parentheses, when you can omit types, and when you need braces and a `return` statement.

### Purposes

- To provide a consistent, readable notation for anonymous functions
- To allow the compiler to infer parameter types when possible
- To support both expression-style and block-style lambda bodies

### Syntax Rules and Structure

**Complete General Syntax:**

```
// Form 1: Expression body
(parameters) -> expression

// Form 2: Block body
(parameters) -> { statements; }

// Form 3: Single inferred parameter (parentheses optional)
parameter -> expression

// Form 4: No parameters
() -> expression
```

**Component Breakdown:**

| Form | Example | Notes |
|------|---------|-------|
| Expression body | `(a, b) -> a + b` | `return` is implicit |
| Block body | `(a, b) -> { return a + b; }` | `return` is explicit |
| Single inferred parameter | `x -> x * 2` | Parentheses omitted |
| No parameters | `() -> 42` | Empty parentheses required |

**Syntax Rules:**

- If the lambda body is a single expression, it is treated as a return value (for non-void functional interfaces).
- If the lambda body is a block, it must be enclosed in braces; a `return` statement is required for non-void functional interfaces.
- Parameter types may be omitted if they are inferable from the target type.
- A lambda expression can be explicitly typed: `(int a, int b) -> a + b`.
- The arrow token `->` cannot be omitted.

**Constraints and Limitations:**

- A lambda expression cannot declare type parameters.
- A lambda expression cannot have a `throws` clause.
- A lambda expression cannot be used where a class instance is required (only functional interfaces).
- If a lambda body is a block, it must be value-compatible or void-compatible.

### Annotated Complete Step-by-Step Code Examples

**Example 1: All Lambda Syntax Forms**

```java
// File: LambdaSyntaxForms.java
public class LambdaSyntaxForms {
    public static void main(String[] args) {
        // Functional interfaces
        interface NoArg { String get(); }
        interface OneArg { int apply(int x); }
        interface TwoArg { int apply(int a, int b); }

        // Form 4: No parameters
        NoArg noArg = () -> "Hello";
        System.out.println(noArg.get());  // Output: Hello

        // Form 3: Single inferred parameter (parentheses omitted)
        OneArg doubleIt = x -> x * 2;
        System.out.println(doubleIt.apply(5));  // Output: 10

        // Form 1: Expression body with two parameters
        TwoArg add = (a, b) -> a + b;
        System.out.println(add.apply(3, 4));  // Output: 7

        // Form 2: Block body with explicit return
        TwoArg multiply = (a, b) -> {
            int result = a * b;
            return result;
        };
        System.out.println(multiply.apply(3, 4));  // Output: 12

        // Explicitly typed parameters
        TwoArg subtract = (int a, int b) -> a - b;
        System.out.println(subtract.apply(10, 3));  // Output: 7
    }
}
```

**Expected Output:**
```
Hello
10
7
12
7
```

**Step-by-Step Explanation:**

1. `NoArg noArg = () -> "Hello"` uses empty parentheses because there are no parameters.
2. `OneArg doubleIt = x -> x * 2` omits parentheses because there is a single inferred parameter.
3. `TwoArg add = (a, b) -> a + b` uses an expression body; the result is returned implicitly.
4. `TwoArg multiply` uses a block body with an explicit `return`.
5. `TwoArg subtract` explicitly declares parameter types, which is optional but valid.

### Real-World Cases

- **Comparator:** `(String a, String b) -> a.length() - b.length()` — explicitly typed parameters.
- **Runnable:** `() -> System.out.println("Running")` — no parameters.
- **Predicate:** `s -> s.isEmpty()` — single inferred parameter.

---

## 3. Functional Interfaces

### Definitions

**Core Definition:** A functional interface is an interface that contains exactly one abstract method (aside from methods of `Object`), representing a single function contract.

**Technical Definition:** "A functional interface is an interface that has just one abstract method (aside from the methods of Object), and thus represents a single function contract". "Functional interfaces provide target types for lambda expressions and method references. Each functional interface has a single abstract method, called the functional method for that functional interface, to which the lambda expression's parameter and return types are matched or adapted".

**Beginner-Friendly Explanation:** A functional interface is like a job description with exactly one task. Anyone (a lambda) who can do that one task qualifies for the job. The interface defines what the task is; the lambda provides how to do it.

### Purposes

- To provide a target type for lambda expressions and method references
- To encapsulate a single unit of behaviour as an object
- To enable functional programming patterns in Java
- To standardise common function shapes in the `java.util.function` package

### Syntax Rules and Structure

**Complete General Syntax:**

```java
@FunctionalInterface
public interface InterfaceName {
    returnType methodName(parameters);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@FunctionalInterface` | Optional annotation that captures design intent and helps the compiler identify violations. |
| Single abstract method | Exactly one abstract method; default and static methods are allowed. |
| Functional method | The single abstract method to which lambda parameters and return types are matched. |

**Syntax Rules:**

- A functional interface must have exactly one abstract method (not counting `Object` methods).
- It may have any number of default and static methods.
- The `@FunctionalInterface` annotation is not required but is recommended.
- Lambda expressions and method references can instantiate functional interfaces.

**Constraints and Limitations:**

- An interface with zero abstract methods is not functional.
- An interface with two or more abstract methods is not functional.
- Methods of `Object` (e.g., `equals`, `hashCode`, `toString`) do not count toward the abstract method count.
- Lambdas cannot instantiate abstract classes, only functional interfaces.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Custom Functional Interface**

```java
// File: FunctionalInterfaceDemo.java
public class FunctionalInterfaceDemo {

    // Step 1: Define a functional interface with @FunctionalInterface
    @FunctionalInterface
    interface MathOperation {
        int operate(int a, int b);   // Single abstract method

        // Default method — allowed
        default int operateWithMessage(int a, int b) {
            System.out.println("Operating on " + a + " and " + b);
            return operate(a, b);
        }

        // Static method — allowed
        static MathOperation addition() {
            return (a, b) -> a + b;
        }
    }

    public static void main(String[] args) {
        // Step 2: Implement the functional interface with lambdas
        MathOperation add = (a, b) -> a + b;
        MathOperation multiply = (a, b) -> a * b;

        System.out.println("Add: " + add.operate(3, 4));        // Output: Add: 7
        System.out.println("Multiply: " + multiply.operate(3, 4)); // Output: Multiply: 12

        // Step 3: Use the default method
        System.out.println("With message: " + add.operateWithMessage(5, 6));
        // Output: Operating on 5 and 6
        //         With message: 11

        // Step 4: Use the static factory method
        MathOperation add2 = MathOperation.addition();
        System.out.println("Add2: " + add2.operate(10, 20));    // Output: Add2: 30
    }
}
```

**Expected Output:**
```
Add: 7
Multiply: 12
Operating on 5 and 6
With message: 11
Add2: 30
```

**Step-by-Step Explanation:**

1. `MathOperation` is annotated with `@FunctionalInterface` and has exactly one abstract method `operate`.
2. The `default` method `operateWithMessage` and the `static` method `addition` are allowed because they are not abstract.
3. The lambdas `(a, b) -> a + b` and `(a, b) -> a * b` implement the single abstract method.
4. `add.operateWithMessage(5, 6)` calls the default method, which internally calls `operate(5, 6)`.
5. `MathOperation.addition()` returns a lambda that adds two numbers.

**Example 2: Standard Functional Interfaces from `java.util.function`**

```java
// File: StandardFunctionalInterfaces.java
import java.util.function.*;

public class StandardFunctionalInterfaces {
    public static void main(String[] args) {
        // Predicate<T>: T -> boolean
        Predicate<String> isEmpty = s -> s.isEmpty();
        System.out.println("Is empty: " + isEmpty.test(""));       // Output: Is empty: true

        // Function<T, R>: T -> R
        Function<String, Integer> length = s -> s.length();
        System.out.println("Length: " + length.apply("Hello"));    // Output: Length: 5

        // Consumer<T>: T -> void
        Consumer<String> printer = s -> System.out.println("Value: " + s);
        printer.accept("Java");                                     // Output: Value: Java

        // Supplier<T>: () -> T
        Supplier<Double> random = () -> Math.random();
        System.out.println("Random: " + (random.get() >= 0));      // Output: Random: true

        // BiFunction<T, U, R>: (T, U) -> R
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        System.out.println("Sum: " + add.apply(3, 4));             // Output: Sum: 7

        // UnaryOperator<T>: T -> T
        UnaryOperator<String> upper = s -> s.toUpperCase();
        System.out.println("Upper: " + upper.apply("hello"));      // Output: Upper: HELLO

        // BinaryOperator<T>: (T, T) -> T
        BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
        System.out.println("Max: " + max.apply(10, 20));           // Output: Max: 20
    }
}
```

**Expected Output:**
```
Is empty: true
Length: 5
Value: Java
Random: true
Sum: 7
Upper: HELLO
Max: 20
```

**Step-by-Step Explanation:**

1. `Predicate<String>` takes a `String` and returns a `boolean`. The lambda `s -> s.isEmpty()` implements `test`.
2. `Function<String, Integer>` takes a `String` and returns an `Integer`. The lambda `s -> s.length()` implements `apply`.
3. `Consumer<String>` takes a `String` and returns nothing. The lambda `s -> System.out.println(...)` implements `accept`.
4. `Supplier<Double>` takes no arguments and returns a `Double`. The lambda `() -> Math.random()` implements `get`.
5. `BiFunction<Integer, Integer, Integer>` takes two arguments and returns a result.
6. `UnaryOperator<String>` is a `Function` where input and output types are the same.
7. `BinaryOperator<Integer>` is a `BiFunction` where all three types are the same.

### Real-World Cases

- **Stream API:** `Predicate` is used in `filter()`, `Function` in `map()`, `Consumer` in `forEach()`.
- **Optional:** `Optional.map(Function)`, `Optional.filter(Predicate)`, `Optional.ifPresent(Consumer)`.
- **CompletableFuture:** `thenApply(Function)`, `thenAccept(Consumer)`, `thenRun(Runnable)`.

---

## 4. Target Typing

### Definitions

**Core Definition:** Target typing is the compiler's ability to infer the type of a lambda expression from the context in which it appears.

**Technical Definition:** "A lambda expression is compatible in an assignment context, invocation context, or casting context with a target type T if T is a functional interface type". "If a lambda expression is implicitly typed, then its lambda body is interpreted according to the context in which it appears".

**Beginner-Friendly Explanation:** Target typing means the compiler figures out what type of lambda you're writing based on where you're using it. If you assign a lambda to a `Predicate<String>`, the compiler knows the lambda takes a `String` and returns a `boolean`.

### Purposes

- To allow lambda parameter types to be omitted, making code more concise
- To enable the same lambda syntax to work with different functional interfaces
- To support overload resolution when lambdas are passed to overloaded methods

### Syntax Rules and Structure

**Complete General Syntax:**

```
TargetType variable = lambdaExpression;
```

**Component Breakdown:**

| Context | Example |
|---------|---------|
| Assignment context | `Predicate<String> p = s -> s.isEmpty();` |
| Method invocation context | `list.removeIf(s -> s.isEmpty());` |
| Cast context | `(Predicate<String>) s -> s.isEmpty()` |

**Syntax Rules:**

- The target type must be a functional interface.
- The compiler derives the function type from the target type and checks congruence.
- If the lambda is explicitly typed, its parameter types must exactly match the function type's parameter types.
- If the lambda is implicitly typed, the compiler infers parameter types from the function type.

**Constraints and Limitations:**

- A lambda expression cannot appear in a context without a target type (e.g., `var x = () -> 42;` is illegal).
- The target type cannot be an abstract class; it must be a functional interface.
- Ambiguous target types cause compile-time errors.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Target Typing in Different Contexts**

```java
// File: TargetTypingDemo.java
import java.util.function.*;

public class TargetTypingDemo {
    public static void main(String[] args) {
        // Context 1: Assignment context
        // The compiler infers that the lambda takes a String and returns a boolean
        Predicate<String> isEmpty = s -> s.isEmpty();
        System.out.println(isEmpty.test(""));       // Output: true
        System.out.println(isEmpty.test("Hello"));  // Output: false

        // Context 2: Method invocation context
        // The compiler infers the lambda type from the method parameter
        printResult(s -> s.length() > 5, "Hello World");
        // Output: Result: true

        // Context 3: Cast context
        // The compiler uses the cast to determine the target type
        Object obj = (Function<String, Integer>) s -> s.length();
        Function<String, Integer> length = (Function<String, Integer>) obj;
        System.out.println("Length: " + length.apply("Java"));  // Output: Length: 4
    }

    // The parameter type Predicate<String> provides the target type
    static void printResult(Predicate<String> predicate, String value) {
        System.out.println("Result: " + predicate.test(value));
    }
}
```

**Expected Output:**
```
true
false
Result: true
Length: 4
```

**Step-by-Step Explanation:**

1. In the assignment context, `Predicate<String> isEmpty = s -> s.isEmpty()` — the compiler knows `s` is a `String` because the target type is `Predicate<String>`.
2. In the method invocation context, `printResult(s -> s.length() > 5, "Hello World")` — the compiler infers the lambda type from the `Predicate<String>` parameter of `printResult`.
3. In the cast context, the cast `(Function<String, Integer>)` tells the compiler the target type.
4. In all cases, the parameter type `s` is inferred, and the return type is checked for compatibility.

**Example 2: Overload Resolution with Target Typing**

```java
// File: OverloadTargetTyping.java
import java.util.function.*;

public class OverloadTargetTyping {
    // Overloaded method: one takes Function, the other takes Supplier
    static void process(Function<String, String> func) {
        System.out.println("Function: " + func.apply("input"));
    }

    static void process(Supplier<String> supplier) {
        System.out.println("Supplier: " + supplier.get());
    }

    public static void main(String[] args) {
        // The target type is Function<String, String>
        process(s -> s.toUpperCase());
        // Output: Function: INPUT

        // The target type is Supplier<String>
        process(() -> "Hello from supplier");
        // Output: Supplier: Hello from supplier
    }
}
```

**Expected Output:**
```
Function: INPUT
Supplier: Hello from supplier
```

**Step-by-Step Explanation:**

1. Two overloaded `process` methods exist: one takes `Function<String, String>` and the other takes `Supplier<String>`.
2. `process(s -> s.toUpperCase())` — the lambda takes one parameter, so it cannot match `Supplier` (which takes zero parameters). The compiler selects the `Function` overload.
3. `process(() -> "Hello from supplier")` — the lambda takes zero parameters, so it matches `Supplier`.
4. Target typing enables the compiler to resolve overloads based on lambda arity (number of parameters).

### Real-World Cases

- **Collections.sort:** `list.sort((a, b) -> a.compareTo(b))` — target type is `Comparator<T>`.
- **Stream.map:** `stream.map(s -> s.toUpperCase())` — target type is `Function<T, R>`.
- **Optional.ifPresent:** `optional.ifPresent(v -> System.out.println(v))` — target type is `Consumer<T>`.

---

## 5. Lambda Scope

### Definitions

**Core Definition:** Lambda scope refers to the region of code in which a lambda expression's parameters and body are visible, and how names inside the lambda relate to names outside it.

**Technical Definition:** "Lambda expressions are lexically scoped. This means that code inside a lambda sees only the symbols as the code immediately outside the lambda expression". "A lambda expression does not introduce a new level of scoping".

**Beginner-Friendly Explanation:** A lambda is like a window into the surrounding code. It can see everything that the surrounding code can see, and `this` inside a lambda refers to the same object as `this` outside the lambda. In contrast, an anonymous class creates a new scope with its own `this`.

### Purposes

- To allow lambdas to access variables from the enclosing method
- To maintain consistent meaning of `this` between the lambda and its enclosing code
- To simplify the mental model of variable visibility

### Syntax Rules and Structure

**Complete General Syntax:**

```
// A lambda can access:
// - Its own parameters
// - Local variables of the enclosing method (if final or effectively final)
// - Instance variables of the enclosing class
// - Static variables of the enclosing class
```

**Component Breakdown:**

| Variable Type | Accessible? | Constraint |
|---------------|-------------|------------|
| Lambda parameters | Yes | Local to the lambda |
| Enclosing local variables | Yes | Must be final or effectively final |
| Instance variables | Yes | Always accessible |
| Static variables | Yes | Always accessible |
| `this` | Refers to enclosing class instance | Not the lambda itself |

**Syntax Rules:**

- Lambda expressions are lexically scoped — they do not introduce a new scope.
- `this` inside a lambda refers to the enclosing class instance, not the lambda.
- Lambda parameters cannot shadow local variables of the enclosing method.
- Local variables referenced from a lambda must be final or effectively final.

**Constraints and Limitations:**

- A lambda cannot modify a local variable from the enclosing scope.
- A lambda cannot declare a variable with the same name as a local variable in the enclosing scope.
- The `super` keyword inside a lambda refers to the enclosing class's superclass.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Lambda Scope and `this`**

```java
// File: LambdaScopeDemo.java
public class LambdaScopeDemo {
    private String instanceVar = "Instance Variable";

    public void demonstrateScope() {
        String localVar = "Local Variable";   // Effectively final

        // Lambda can access instance variables and local variables
        Runnable r = () -> {
            System.out.println("Instance: " + instanceVar);   // OK
            System.out.println("Local: " + localVar);         // OK
            // System.out.println(this);                     // 'this' refers to LambdaScopeDemo
        };

        r.run();
        // Output: Instance: Instance Variable
        //         Local: Local Variable
    }

    public static void main(String[] args) {
        new LambdaScopeDemo().demonstrateScope();
    }
}
```

**Expected Output:**
```
Instance: Instance Variable
Local: Local Variable
```

**Step-by-Step Explanation:**

1. `instanceVar` is an instance variable of `LambdaScopeDemo`. Lambdas can always access instance variables.
2. `localVar` is a local variable in `demonstrateScope`. It is effectively final (assigned once, never modified), so the lambda can capture it.
3. If `this` were used inside the lambda, it would refer to the `LambdaScopeDemo` instance, not a new scope.

**Example 2: Effectively Final Constraint**

```java
// File: EffectivelyFinalDemo.java
public class EffectivelyFinalDemo {
    public static void main(String[] args) {
        // Example 1: Effectively final (valid)
        int effectivelyFinal = 10;
        Runnable r1 = () -> System.out.println("Value: " + effectivelyFinal);
        r1.run();  // Output: Value: 10

        // Example 2: Not effectively final (compile error)
        int notFinal = 10;
        notFinal = 20;  // Reassignment makes it not effectively final
        // Runnable r2 = () -> System.out.println(notFinal);  // COMPILE ERROR!

        // Workaround: use a final copy
        int finalCopy = notFinal;
        Runnable r3 = () -> System.out.println("Copy: " + finalCopy);
        r3.run();  // Output: Copy: 20
    }
}
```

**Expected Output:**
```
Value: 10
Copy: 20
```

**Step-by-Step Explanation:**

1. `effectivelyFinal` is assigned once and never modified, so it is effectively final. The lambda can capture it.
2. `notFinal` is reassigned, so it is not effectively final. The compiler would reject a lambda that captures it.
3. The workaround creates a `finalCopy` of the value, which is effectively final and can be captured.

### Real-World Cases

- **Event Handlers:** A lambda for a button click can access instance variables of the controller class.
- **Stream Pipelines:** Lambdas in `filter()` and `map()` can reference effectively final variables from the enclosing method.
- **Callbacks:** A lambda passed to an asynchronous method can capture final configuration values.

---

## 6. Lambda Capture

### Definitions

**Core Definition:** Lambda capture is the mechanism by which a lambda expression accesses variables from its enclosing lexical scope.

**Technical Definition:** "If a lambda expression accesses a local variable defined in the enclosing method, that variable must be final or effectively final". "Captured variables work by copying their values".

**Beginner-Friendly Explanation:** When a lambda uses a variable from outside itself, it "captures" that variable by copying its value. This is like taking a snapshot — the lambda gets its own copy of the value, so later changes to the original variable don't affect the lambda.

### Purposes

- To allow lambdas to use data from the surrounding context
- To ensure thread safety by preventing modification of captured variables
- To provide closure-like behaviour in Java

### Syntax Rules and Structure

**Complete General Syntax:**

```
int x = 10;                    // Effectively final
Runnable r = () -> {
    System.out.println(x);     // Captures x by value
};
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| Captured variable | A local variable from the enclosing scope used inside the lambda. |
| Copy semantics | The variable's value is copied at the time the lambda is created. |
| Effectively final | The variable must not be reassigned after initialisation. |

**Syntax Rules:**

- Only final or effectively final local variables can be captured.
- Instance variables and static variables are not subject to the effectively final constraint.
- Captured variables are copied; modifications inside the lambda affect the copy, not the original (but modification is not allowed for local variables).

**Constraints and Limitations:**

- A lambda cannot modify a captured local variable.
- A captured variable's value is fixed at the time the lambda is created.
- Loop variables in basic `for` loops are not effectively final and cannot be captured (though enhanced `for` loop variables are new per iteration and can be captured).

### Annotated Complete Step-by-Step Code Examples

**Example 1: Capturing a Local Variable**

```java
// File: CaptureDemo.java
public class CaptureDemo {
    public static void main(String[] args) {
        String message = "Hello from captured variable";  // Effectively final

        Runnable r = () -> {
            // The lambda captures 'message' by copying its value
            System.out.println(message);
        };

        r.run();
        // Output: Hello from captured variable

        // The original variable is unchanged (and cannot be changed if captured)
        System.out.println("Original: " + message);
        // Output: Original: Hello from captured variable
    }
}
```

**Expected Output:**
```
Hello from captured variable
Original: Hello from captured variable
```

**Step-by-Step Explanation:**

1. `message` is a local variable assigned once, making it effectively final.
2. The lambda `() -> System.out.println(message)` captures `message` by copying its value.
3. When `r.run()` is called, the lambda prints the captured value.
4. The original `message` is unaffected by the lambda.

**Example 2: Capturing in a Loop**

```java
// File: LoopCaptureDemo.java
import java.util.*;

public class LoopCaptureDemo {
    public static void main(String[] args) {
        // Enhanced for loop: each iteration creates a new variable
        List<String> names = List.of("Alice", "Bob", "Charlie");
        List<Runnable> tasks = new ArrayList<>();

        for (String name : names) {
            // 'name' is a new variable each iteration — effectively final
            tasks.add(() -> System.out.println("Hello, " + name));
        }

        // Run all tasks
        for (Runnable task : tasks) {
            task.run();
        }
        // Output: Hello, Alice
        //         Hello, Bob
        //         Hello, Charlie

        // Basic for loop: the loop variable is NOT effectively final
        // The following would cause a compile error:
        // for (int i = 0; i < 3; i++) {
        //     tasks.add(() -> System.out.println(i));  // ERROR!
        // }
    }
}
```

**Expected Output:**
```
Hello, Alice
Hello, Bob
Hello, Charlie
```

**Step-by-Step Explanation:**

1. In the enhanced `for` loop, `name` is a new variable in each iteration, so it is effectively final within each iteration.
2. Each lambda captures the `name` value for its specific iteration.
3. When the tasks are run later, each prints the correct name.
4. In a basic `for` loop, `i` is incremented, making it not effectively final, so it cannot be captured.

### Real-World Cases

- **Stream Operations:** Lambdas in `forEach` capture the current element from the stream.
- **Event Listeners:** A lambda captures configuration values from the enclosing method.
- **Thread Pools:** Tasks submitted to an executor capture final parameters.

---

## 7. Effectively Final

### Definitions

**Core Definition:** A variable is effectively final if it is not declared `final` but is never reassigned after initialisation.

**Technical Definition:** "A local variable or a method, constructor, lambda, or exception parameter is effectively final if it is not final but it never occurs as the left-hand operand of an assignment operator or as the operand of a prefix or postfix increment or decrement operator".

**Beginner-Friendly Explanation:** A variable is effectively final if you treat it like it's final — you assign it once and never change it. Even without the `final` keyword, the compiler treats it as final for the purposes of lambda capture.

### Purposes

- To allow lambda expressions to capture local variables without requiring explicit `final` declarations
- To ensure that captured values are stable and thread-safe
- To simplify code by avoiding unnecessary `final` keywords

### Syntax Rules and Structure

**Complete General Syntax:**

```
int x = 10;        // Effectively final — assigned once
x = 20;            // Now NOT effectively final — reassigned
```

**Component Breakdown:**

| Condition | Effectively Final? |
|-----------|-------------------|
| Assigned once, never modified | Yes |
| Declared `final` | Yes |
| Reassigned after initialisation | No |
| Incremented or decremented | No |
| Modified in a loop header | No (for basic for loop) |

**Syntax Rules:**

- A variable is effectively final if it is assigned exactly once and never modified afterward.
- The `final` keyword is not required, but it guarantees effectively final status.
- Captured variables must be final or effectively final.
- Instance and static variables are not subject to this constraint.

**Constraints and Limitations:**

- Effectively final is a compile-time concept; the compiler checks it.
- A variable that "might as well" be final but is reassigned later cannot be captured.
- The workaround for non-effectively-final variables is to create a final copy.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Effectively Final vs Not Effectively Final**

```java
// File: EffectivelyFinalCheck.java
public class EffectivelyFinalCheck {
    public static void main(String[] args) {
        // Case 1: Effectively final (valid)
        int a = 5;
        Runnable r1 = () -> System.out.println(a);
        r1.run();  // Output: 5

        // Case 2: Explicitly final (valid)
        final int b = 10;
        Runnable r2 = () -> System.out.println(b);
        r2.run();  // Output: 10

        // Case 3: Not effectively final (compile error)
        int c = 15;
        c = 20;  // Reassignment
        // Runnable r3 = () -> System.out.println(c);  // COMPILE ERROR!

        // Case 4: Workaround — copy to final variable
        final int d = c;  // d is effectively final
        Runnable r4 = () -> System.out.println(d);
        r4.run();  // Output: 20

        // Case 5: Loop variable (enhanced for — valid)
        for (String s : new String[]{"X", "Y"}) {
            Runnable r5 = () -> System.out.println(s);
            r5.run();  // Output: X, then Y
        }
    }
}
```

**Expected Output:**
```
5
10
20
X
Y
```

**Step-by-Step Explanation:**

1. `a` is assigned once and never modified — effectively final. The lambda captures it.
2. `b` is explicitly `final` — always valid for capture.
3. `c` is reassigned, so it is not effectively final. The compiler rejects the lambda.
4. `d` is a final copy of `c`'s current value, making it effectively final.
5. In the enhanced `for` loop, `s` is a new variable each iteration, so it is effectively final within each iteration.

### Real-World Cases

- **Stream Pipelines:** Variables defined before a stream operation and used in a lambda must be effectively final.
- **Callbacks:** Configuration values captured by callbacks are often declared final or effectively final.
- **Concurrency:** Effectively final variables are safe to capture because their values cannot change.

---

## 8. Lambda vs Anonymous Classes

### Definitions

**Core Definition:** Lambda expressions and anonymous classes are both mechanisms for creating instances of interfaces or abstract classes inline, but they differ in syntax, scope, and capabilities.

**Technical Definition:** "Anonymous class: Use it if you need to declare fields or additional methods. Use it if you are encapsulating a single unit of behavior that you want to pass to other code. Use it if you need a simple instance of a functional interface and none of the preceding criteria apply". "Lambdas are limited to functional interfaces. If you want to create an instance of an abstract class, you can do it with an anonymous class, but not a lambda".

**Beginner-Friendly Explanation:** An anonymous class is like a full recipe card with a name, ingredients, and instructions. A lambda is like a sticky note with just the essential instruction. The sticky note is shorter and simpler, but it can't do everything a full recipe card can.

### Purposes

- To provide a simpler syntax for implementing functional interfaces
- To reduce boilerplate code compared to anonymous classes
- To enable functional programming patterns
- To choose the right tool for the job (lambda for simple behaviour, anonymous class for complex)

### Syntax Rules and Structure

**Complete General Syntax (Lambda):**

```
(parameters) -> expression
```

**Complete General Syntax (Anonymous Class):**

```java
new InterfaceName() {
    @Override
    public returnType methodName(parameters) {
        // implementation
    }
};
```

**Component Breakdown:**

| Feature | Lambda | Anonymous Class |
|---------|--------|-----------------|
| Target | Functional interface only | Any interface or abstract class |
| Syntax | `(a, b) -> a + b` | `new Interface() { public int method(int a, int b) { return a + b; } }` |
| Scope | Lexically scoped (no new scope) | Introduces a new scope |
| `this` | Refers to enclosing instance | Refers to anonymous class instance |
| Fields | Cannot declare fields | Can declare fields |
| Methods | Cannot declare additional methods | Can declare additional methods |
| State | Stateless | Can maintain state |
| Serialization | Not reliably serializable | Can be serializable |

**Syntax Rules:**

- Lambdas can only target functional interfaces.
- Anonymous classes can target any interface or abstract class.
- Lambdas cannot declare fields or additional methods.
- Anonymous classes can declare fields, additional methods, and instance initialisers.
- Lambdas are lexically scoped; anonymous classes introduce a new scope.

**Constraints and Limitations:**

- Lambdas cannot be used where an abstract class instance is required.
- Lambdas cannot maintain state across invocations (unless captured from effectively final variables).
- Anonymous classes are more verbose but more flexible.
- "Prefer lambdas to anonymous classes" (Effective Java, Item 42).

### Annotated Complete Step-by-Step Code Examples

**Example 1: Same Behaviour — Lambda vs Anonymous Class**

```java
// File: LambdaVsAnonymous.java
import java.util.*;

public class LambdaVsAnonymous {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>(List.of("Charlie", "Alice", "Bob"));

        // Approach 1: Anonymous class for sorting
        Collections.sort(names, new Comparator<String>() {
            @Override
            public int compare(String a, String b) {
                return a.compareTo(b);
            }
        });
        System.out.println("Anonymous sort: " + names);
        // Output: Anonymous sort: [Alice, Bob, Charlie]

        // Approach 2: Lambda expression for sorting
        List<String> names2 = new ArrayList<>(List.of("Charlie", "Alice", "Bob"));
        Collections.sort(names2, (a, b) -> a.compareTo(b));
        System.out.println("Lambda sort: " + names2);
        // Output: Lambda sort: [Alice, Bob, Charlie]
    }
}
```

**Expected Output:**
```
Anonymous sort: [Alice, Bob, Charlie]
Lambda sort: [Alice, Bob, Charlie]
```

**Step-by-Step Explanation:**

1. The anonymous class implements `Comparator<String>` with a full method declaration.
2. The lambda expression `(a, b) -> a.compareTo(b)` achieves the same result with far less code.
3. Both approaches produce the same sorted output.
4. The lambda is more concise and readable for this simple behaviour.

**Example 2: When Anonymous Class is Necessary**

```java
// File: AnonymousNecessary.java
public class AnonymousNecessary {
    // Abstract class — cannot be instantiated with a lambda
    abstract static class Greeter {
        abstract void greet();
        void greetTwice() {
            greet();
            greet();
        }
    }

    public static void main(String[] args) {
        // Lambda cannot be used here because Greeter is an abstract class
        // Must use an anonymous class
        Greeter greeter = new Greeter() {
            @Override
            void greet() {
                System.out.println("Hello!");
            }

            // Can add additional methods and fields
            private int count = 0;
            void increment() { count++; }
        };

        greeter.greetTwice();
        // Output: Hello!
        //         Hello!
    }
}
```

**Expected Output:**
```
Hello!
Hello!
```

**Step-by-Step Explanation:**

1. `Greeter` is an abstract class, not a functional interface.
2. Lambdas can only instantiate functional interfaces, so an anonymous class is required.
3. The anonymous class can add fields (`count`) and methods (`increment`), which lambdas cannot do.
4. The `greetTwice()` method inherited from `Greeter` calls `greet()` twice.

### Real-World Cases

- **Event Listeners:** Simple listeners can use lambdas; listeners needing state or multiple methods require anonymous classes.
- **Threads:** `new Thread(() -> ...)` uses a lambda; a thread that needs custom fields uses an anonymous class.
- **Comparators:** Simple comparisons use lambdas; complex comparators with state use anonymous classes.

---

## 9. Lambda Best Practices

### Definitions

**Core Definition:** Lambda best practices are established guidelines for writing effective, readable, and maintainable lambda expressions.

**Technical Definition:** Best practices derive from authoritative sources including *Effective Java* by Joshua Bloch, the JDK's own coding style guide, and industry experience. Key principles include keeping lambdas short, preferring method references, using standard functional interfaces, and avoiding complex lambda bodies.

**Beginner-Friendly Explanation:** Best practices are like cooking tips — they help you avoid common mistakes and write better code. For lambdas, the key is to keep them simple, use method references when possible, and don't try to do too much in a single lambda.

### Purposes

- To improve code readability and maintainability
- To reduce the likelihood of bugs
- To leverage standard library features effectively
- To ensure consistent style across a codebase

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Prefer Lambdas to Anonymous Classes | Lambdas are more concise and readable for functional interfaces (Effective Java, Item 42). |
| Prefer Method References to Lambdas | When a lambda only calls an existing method, use a method reference (Effective Java, Item 43). |
| Keep Lambdas Short | "Anonymous lambdas should usually be limited to a couple lines of body code. More complex lambdas should be named". |
| Use Standard Functional Interfaces | Prefer `java.util.function` interfaces over custom ones (Effective Java, Item 44). |
| Avoid Complex Lambda Bodies | Extract complex logic into a separate method and call it from the lambda. |
| Use Meaningful Parameter Names | Even though types are inferred, parameter names should convey meaning. |
| Avoid Capturing Unnecessary Variables | Capturing fewer variables reduces memory overhead and improves clarity. |
| Don't Use Lambdas in Loops | Creating lambdas in loops can cause performance issues; consider extracting. |

**Syntax Rules:**

- If a lambda body exceeds two or three lines, consider extracting it to a named method.
- Use method references (`String::length`) instead of lambdas (`s -> s.length()`) when possible.
- Use standard functional interfaces from `java.util.function` whenever they fit.
- Name lambda parameters descriptively, even when types are inferred.

**Constraints and Limitations:**

- Best practices are guidelines, not absolute rules; context matters.
- Over-application of "keep lambdas short" can lead to excessive method extraction.
- Performance-critical code may benefit from avoiding lambda allocation in hot loops.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Prefer Method References**

```java
// File: MethodRefBestPractice.java
import java.util.*;
import java.util.function.*;

public class MethodRefBestPractice {
    public static void main(String[] args) {
        List<String> names = List.of("Alice", "Bob", "Charlie");

        // BAD: Lambda that just calls an existing method
        names.forEach(s -> System.out.println(s));

        // GOOD: Method reference — more concise and readable
        names.forEach(System.out::println);

        // BAD: Lambda with unnecessary parameter and method call
        Function<String, Integer> badLength = s -> s.length();

        // GOOD: Method reference
        Function<String, Integer> goodLength = String::length;
        System.out.println("Length: " + goodLength.apply("Hello"));  // Output: 5

        // BAD: Complex lambda body
        List<Integer> lengths = new ArrayList<>();
        names.forEach(s -> {
            int len = s.length();
            if (len > 3) {
                lengths.add(len);
            }
        });

        // GOOD: Extract to a method and use a method reference
        names.stream()
             .map(String::length)
             .filter(len -> len > 3)
             .forEach(lengths::add);
        System.out.println("Lengths > 3: " + lengths);
        // Output: Lengths > 3: [5, 7]
    }
}
```

**Expected Output:**
```
Alice
Bob
Charlie
Length: 5
Lengths > 3: [5, 7]
```

**Step-by-Step Explanation:**

1. `s -> System.out.println(s)` is replaced by `System.out::println`, a method reference.
2. `s -> s.length()` is replaced by `String::length`.
3. The complex lambda body in the `forEach` is replaced by a stream pipeline with `map`, `filter`, and `forEach` using method references.
4. Method references and stream operations are more readable and maintainable.

**Example 2: Keep Lambdas Short**

```java
// File: ShortLambdaBestPractice.java
import java.util.*;
import java.util.function.*;

public class ShortLambdaBestPractice {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "cherry", "date");

        // BAD: Complex lambda with multiple statements
        Predicate<String> badPredicate = s -> {
            String trimmed = s.trim();
            if (trimmed.isEmpty()) {
                return false;
            }
            int length = trimmed.length();
            return length > 3 && length < 10;
        };

        // GOOD: Extract logic to a named method
        Predicate<String> goodPredicate = ShortLambdaBestPractice::isValidWord;

        // Both produce the same result
        words.stream()
             .filter(goodPredicate)
             .forEach(System.out::println);
        // Output: apple
        //         banana
        //         cherry

        // Another good practice: use standard functional interfaces
        Function<String, Integer> length = String::length;       // standard
        // Not: interface MyFunction { int getLength(String s); }
    }

    // Named method extracted from the complex lambda
    static boolean isValidWord(String s) {
        String trimmed = s.trim();
        if (trimmed.isEmpty()) {
            return false;
        }
        int length = trimmed.length();
        return length > 3 && length < 10;
    }
}
```

**Expected Output:**
```
apple
banana
cherry
```

**Step-by-Step Explanation:**

1. The `badPredicate` lambda contains multiple statements and complex logic, violating the "keep lambdas short" principle.
2. The `goodPredicate` uses a method reference to a named method `isValidWord`, which contains the same logic but is more readable and reusable.
3. The stream pipeline uses the method reference, and the output is the same.
4. The `Function<String, Integer> length = String::length` uses the standard `Function` interface instead of a custom one.

### Real-World Cases

- **Stream Pipelines:** Best practice is to use method references like `String::toUpperCase` instead of `s -> s.toUpperCase()`.
- **Event Handling:** Simple lambdas like `e -> handleClick(e)` are preferred over anonymous classes.
- **Comparator Chains:** `Comparator.comparing(String::length).thenComparing(String::compareTo)` uses method references and standard interfaces.

---

## References Links

- Oracle Java Tutorials — Lambda Expressions - https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html
- Java Language Specification, Chapter 15: Expressions (Lambda Expressions) - https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.27
- JSR 335: Lambda Expressions for the Java™ Programming Language - https://cr.openjdk.org/~dlsmith/jsr335/jsr335-0.6.2/index.html
- Oracle Java Tutorials — When to Use Nested Classes, Local Classes, Anonymous Classes, and Lambda Expressions - https://docs.oracle.com/javase/tutorial/java/javaOO/whentouse.html
- Java Platform, Standard Edition API Specification — Package java.util.function - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html
- Dev.java — Using Lambdas Expressions in Your Application - https://dev.java/learn/lambdas/
- Effective Java (3rd Edition) by Joshua Bloch — Chapter 7: Lambdas and Streams
- OpenJDK JDK Style Guide — Lambda Expressions - https://github.com/openjdk/jdk/blob/master/doc/hotspot-style.md