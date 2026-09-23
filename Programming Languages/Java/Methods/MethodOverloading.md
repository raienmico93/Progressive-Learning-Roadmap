# Java Method Overloading: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Method overloading** is a feature in Java that allows a class to contain multiple methods with the same name, as long as their parameter lists differ. It enables a single, memorable method name to handle several related variations of an operation.

### Technical Definition

Method overloading is a form of compile-time polymorphism (also called static polymorphism) where the compiler selects which method to invoke based on the number, types, and order of the arguments provided at the call site. Java distinguishes overloaded methods by their **method signatures**, which comprise the method name and the parameter types. The return type and access modifiers are not part of the signature and do not participate in overload resolution.

### Beginner-Friendly Explanation

Imagine a coffee shop where the barista asks, "What size?" You can say "small," "medium," or "large," and the same order ("coffee") gets you different amounts. In Java, you can write one method called `print()` that handles a `String`, an `int`, or a `double`—the compiler figures out which version you mean based on what you pass in.

### Key Characteristics

- **Compile-time resolution**: The compiler chooses which overloaded method to call based on the declared types of the arguments, not their runtime types.
- **Signature-based**: Methods are distinguished by their parameter lists (type, number, and order).
- **Return type is irrelevant**: You cannot overload two methods that differ only in return type.
- **Static binding**: Overloaded method calls are resolved at compile time (static binding), unlike overridden methods which use dynamic binding.
- **Ambiguity risk**: With autoboxing and varargs, overload resolution can become confusing or produce compile-time errors.

### Prerequisites

- Basic Java syntax and method declarations.
- Understanding of method signatures (name + parameter types).
- Familiarity with inheritance and method overriding (to distinguish from overloading).
- Knowledge of primitive types and wrapper classes (for autoboxing concepts).

### Related Programming Areas

- **Object-Oriented Programming**: Overloading is a form of polymorphism (ad-hoc polymorphism).
- **API Design**: Overloaded methods provide convenient interfaces (e.g., `println()`, `valueOf()`).
- **Design Patterns**: Factory methods and builder patterns often use overloading.
- **Compiler Design**: Overload resolution is a key part of type checking and method dispatch.

### Core Concepts / Features

1. Overloaded Methods
2. Parameter Differences (Type, Number, Order)
3. Compile-Time Method Selection (Static Binding)
4. Overloading Rules
5. Autoboxing and Varargs Ambiguities in Overloading Resolution


## Core Concept 1: Overloaded Methods

### Definitions

**Core Definition**: Overloaded methods are two or more methods within the same class (or a class and its subclass) that share the same name but have different parameter lists.

**Technical Definition**: Method overloading occurs when a class declares multiple methods with the same name but different method signatures. The compiler inspects each call to an overloaded method and uses the declared types of the method parameters to decide which method to invoke. Overloaded methods are quite useful but can lead to confusion when combined with autoboxing, generics, or varargs.

**Beginner-Friendly Explanation**: Overloaded methods are like different recipes for the same dish. You have one name—"pasta"—but you can make it with tomato sauce, cream sauce, or pesto. The ingredients you have on hand (the arguments) determine which recipe you follow.

### Purposes

- To provide a single, consistent method name for conceptually similar operations.
- To offer convenience variations of a method without requiring callers to remember different names.
- To support different input types or numbers of inputs with the same logical operation.
- To improve code readability and API usability.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public ReturnType methodName(ParameterList1) { ... }
public ReturnType methodName(ParameterList2) { ... }
// ... where ParameterList1 and ParameterList2 differ in type, number, or order
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methodName` | The shared name of all overloaded versions. |
| `ParameterListN` | Each version must have a unique parameter list. |

**Example:**

```java
public void print(String s) { ... }
public void print(int i) { ... }
public void print(String s, int copies) { ... }
```

**Syntax Rules:**

- The method name must be identical across all overloaded versions.
- The parameter list must differ in at least one of: number of parameters, types of parameters, or order of parameter types.
- Return type may differ freely and does not affect overloading validity.
- Access modifiers may differ freely.
- Overloading can occur within a single class or between a superclass and subclass (a subclass can add new overloads).

**Constraints and Limitations:**

- You cannot overload two methods that differ only in return type.
- You cannot overload two methods that differ only in parameter names.
- Ambiguous overloads can cause compile-time errors.
- Overloading methods with different semantics (e.g., `getData(int)` returns by index, `getData(Integer)` returns by value) is a known anti-pattern.

### Annotated Complete Code Examples

**Example 1: Basic Method Overloading**

```java
/**
 * Demonstrates method overloading with different parameter types and numbers.
 */
public class Printer {

    // Version 1: String parameter
    public void print(String text) {
        System.out.println("String: " + text);
    }

    // Version 2: int parameter
    public void print(int number) {
        System.out.println("int: " + number);
    }

    // Version 3: double parameter
    public void print(double value) {
        System.out.println("double: " + value);
    }

    // Version 4: String and int parameters
    public void print(String text, int copies) {
        for (int i = 0; i < copies; i++) {
            System.out.println("Copy " + (i + 1) + ": " + text);
        }
    }

    public static void main(String[] args) {
        Printer printer = new Printer();

        // The compiler selects the appropriate overload based on argument type
        printer.print("Hello");           // Calls print(String)
        printer.print(42);                // Calls print(int)
        printer.print(3.14);              // Calls print(double)
        printer.print("Java", 3);         // Calls print(String, int)
    }
}
```

**Expected Output:**

```
String: Hello
int: 42
double: 3.14
Copy 1: Java
Copy 2: Java
Copy 3: Java
```

**Why This Output Occurs:**
- Each `print` method has a distinct parameter list: `(String)`, `(int)`, `(double)`, and `(String, int)`.
- The compiler selects the method whose parameter types match the argument types at the call site.
- `print("Hello")` matches `print(String)`; `print(42)` matches `print(int)`; etc.

**Step-by-Step Setup Guide:**
1. Create `Printer.java`.
2. Compile with `javac Printer.java`.
3. Run with `java Printer`.
4. Observe the output.

### Real-World Cases

- **`System.out.println()`**: Overloaded for `String`, `int`, `double`, `boolean`, `char`, `Object`, etc.
- **`String.valueOf()`**: Overloaded for all primitive types and `Object`.
- **`Math.abs()`**: Overloaded for `int`, `long`, `float`, and `double`.
- **Constructors**: `ArrayList()` vs. `ArrayList(int initialCapacity)`.

### References

- Cornell Computer Science – Method Overloading - https://www.cs.cornell.edu/courses/cs100/1999su/P2%20Solutions/p2_5.html
- GUVI – Method Overloading and Method Overriding in Java - https://www.guvi.in/hub/java-oops-concepts-tutorial/method-overloading-and-method-overriding/
- SEI CERT – MET50-J: Avoid ambiguous or confusing uses of overloading - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1946-865/MET50-J.+Avoid+ambiguous+or+conf_da68044bf3914b329fa7d361b584e681-201124-1946-866.pdf


## Core Concept 2: Parameter Differences (Type, Number, and Order)

### Definitions

**Core Definition**: The parameter list of a method comprises the types, number, and order of its parameters. Overloaded methods must differ in at least one of these three dimensions.

**Technical Definition**: Java distinguishes methods by their **method signatures**, which consist of the method name and the parameter types (in order). Two methods with the same name are overloaded if their parameter lists differ in the number of parameters, the types of parameters, or the order of parameter types. The compiler uses the declared types of the arguments at the call site to select the most specific applicable method.

**Beginner-Friendly Explanation**: Think of a method as a lock and the arguments as keys. You can have the same lock name ("open") but different key shapes: a small key (one `int`), a large key (one `double`), or a key with two prongs (two parameters). The shape of the key you insert determines which lock opens.

### Purposes

- To allow a single method name to handle different kinds of inputs.
- To provide flexibility in how callers can invoke a method.
- To support natural, readable API calls with varying argument counts.
- To enable type-specific behavior under a unified name.

### Syntax Rules and Structure

**Complete General Syntax (Parameter Differences):**

```java
// Difference in type
void process(int value) { ... }
void process(String value) { ... }

// Difference in number
void process(int a) { ... }
void process(int a, int b) { ... }

// Difference in order
void process(int a, String b) { ... }
void process(String b, int a) { ... }
```

**Component Breakdown:**

| Dimension | Description | Example |
|-----------|-------------|---------|
| **Type** | Different data types for parameters | `process(int)` vs `process(String)` |
| **Number** | Different count of parameters | `process(int)` vs `process(int, int)` |
| **Order** | Same types, different sequence | `process(int, String)` vs `process(String, int)` |

**Syntax Rules:**

- At least one dimension must differ between overloaded versions.
- The return type does not participate in distinguishing overloads.
- Parameter names do not participate in overload resolution.
- Order-based overloading (same types, different sequence) is legal but often confusing and discouraged.

**Constraints and Limitations:**

- Order-based overloading with the same types in different sequences can create subtle bugs.
- Methods that differ only in parameter names are not valid overloads.
- Overloading can become confusing when combined with autoboxing or varargs.

### Annotated Complete Code Examples

**Example 1: Parameter Differences in Action**

```java
/**
 * Demonstrates overloading by type, number, and order.
 */
public class ParameterDemo {

    // Overloading by TYPE
    public void display(int value) {
        System.out.println("int: " + value);
    }

    public void display(String value) {
        System.out.println("String: " + value);
    }

    // Overloading by NUMBER
    public void display(int a, int b) {
        System.out.println("Two ints: " + a + ", " + b);
    }

    // Overloading by ORDER (same types, different sequence)
    public void display(int a, String b) {
        System.out.println("int then String: " + a + ", " + b);
    }

    public void display(String a, int b) {
        System.out.println("String then int: " + a + ", " + b);
    }

    public static void main(String[] args) {
        ParameterDemo demo = new ParameterDemo();

        demo.display(10);              // TYPE: int
        demo.display("Hello");         // TYPE: String
        demo.display(1, 2);            // NUMBER: two ints
        demo.display(5, "World");      // ORDER: int, String
        demo.display("Java", 7);       // ORDER: String, int
    }
}
```

**Expected Output:**

```
int: 10
String: Hello
Two ints: 1, 2
int then String: 5, World
String then int: Java, 7
```

**Why This Output Occurs:**
- `display(10)` matches `display(int)` by type.
- `display("Hello")` matches `display(String)` by type.
- `display(1, 2)` matches `display(int, int)` by number.
- `display(5, "World")` matches `display(int, String)` by order.
- `display("Java", 7)` matches `display(String, int)` by order.

### Real-World Cases

- **`println()`**: Overloaded by type (`int`, `String`, `double`, etc.).
- **`substring()`**: `substring(int)` vs `substring(int, int)` — overloading by number.
- **Constructors**: `Con(int, String)` vs `Con(String, int)` — overloading by order (discouraged).
- **Builder methods**: `set(int)` vs `set(int, int)` for different configuration granularities.

### References

- Cornell Computer Science – Method Overloading - https://www.cs.cornell.edu/courses/cs100/1999su/P2%20Solutions/p2_5.html
- GUVI – Method Overloading and Method Overriding in Java - https://www.guvi.in/hub/java-oops-concepts-tutorial/method-overloading-and-method-overriding/
- SEI CERT – MET50-J: Avoid ambiguous or confusing uses of overloading - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1946-865/MET50-J.+Avoid+ambiguous+or+conf_da68044bf3914b329fa7d361b584e681-201124-1946-866.pdf


## Core Concept 3: Compile-Time Method Selection (Static Binding)

### Definitions

**Core Definition**: Compile-time method selection (static binding) is the process by which the Java compiler determines which overloaded method to invoke based on the declared types of the arguments at the call site.

**Technical Definition**: In method overloading, the choice of which method to invoke is determined at compile time. The overloaded method associated with the **static type** (declared type) of the arguments is invoked, even when the runtime type differs. This is known as static binding or early binding. The compiler inspects each call to an overloaded method and uses the declared types of the method parameters to decide which method to invoke.

**Beginner-Friendly Explanation**: When you call an overloaded method, the compiler looks at the *type of the variable* you're passing, not the actual object inside it. If you have a variable declared as `Animal` but it holds a `Dog`, and you call an overloaded method that has versions for both `Animal` and `Dog`, the compiler picks the `Animal` version because that's what the variable is declared as. This is different from overriding, where the runtime object type decides.

### Purposes

- To enable the compiler to resolve method calls efficiently at compile time.
- To provide predictable behavior based on declared types.
- To support the overloading mechanism with clear, deterministic selection rules.
- To distinguish overloading (static binding) from overriding (dynamic binding).

### Syntax Rules and Structure

**Compile-Time Selection Rules:**

1. **Phase 1 (Strict Invocation)**: Find applicable methods without boxing/unboxing or varargs.
2. **Phase 2 (Loose Invocation)**: Allow boxing/unboxing but not varargs.
3. **Phase 3 (Variable Arity)**: Allow varargs, boxing, and unboxing.
4. **Most Specific Method**: Among applicable methods, choose the most specific (subtype) parameter types.

**Example of Static Binding:**

```java
class Animal { }
class Dog extends Animal { }

void process(Animal a) { System.out.println("Animal"); }
void process(Dog d) { System.out.println("Dog"); }

Animal a = new Dog(); // Declared type is Animal
process(a); // Compiler selects process(Animal) — static binding
```

**Syntax Rules:**

- Overloaded methods are resolved at compile time using the static types of arguments.
- The runtime type of the object does not affect overload resolution.
- The most specific applicable method is chosen; if none is more specific, the call is ambiguous (compile error).
- Return type is not considered during method selection.

**Constraints and Limitations:**

- Static binding means overloaded methods cannot exhibit polymorphic behavior based on runtime types.
- A common mistake is expecting overloading to behave like overriding (runtime polymorphism).
- Adding a new overload can silently change which method existing code calls (source incompatibility).

### Annotated Complete Code Examples

**Example 1: Static Binding vs. Dynamic Binding**

```java
/**
 * Demonstrates that overloaded methods use static binding (compile-time).
 */
class Animal {
    public void makeNoise() {
        System.out.println("Animal makes noise");
    }
}

class Dog extends Animal {
    @Override
    public void makeNoise() {
        System.out.println("Dog barks");
    }
}

public class BindingDemo {

    // Overloaded methods
    static void process(Animal a) {
        System.out.println("Processing Animal");
    }

    static void process(Dog d) {
        System.out.println("Processing Dog");
    }

    public static void main(String[] args) {
        // Static binding: declared type is Animal
        Animal animalRef = new Dog();
        process(animalRef); // Calls process(Animal) — static binding

        // Static binding: declared type is Dog
        Dog dogRef = new Dog();
        process(dogRef); // Calls process(Dog) — static binding

        // Dynamic binding: overridden method uses runtime type
        animalRef.makeNoise(); // Calls Dog.makeNoise() — dynamic binding
    }
}
```

**Expected Output:**

```
Processing Animal
Processing Dog
Dog barks
```

**Why This Output Occurs:**
- `animalRef` is declared as `Animal`, so the compiler selects `process(Animal)` even though the runtime object is a `Dog`. This is **static binding** for the overloaded method.
- `dogRef` is declared as `Dog`, so the compiler selects `process(Dog)`.
- `animalRef.makeNoise()` uses **dynamic binding**: the JVM looks at the actual runtime type (`Dog`) and invokes `Dog.makeNoise()`.

### Real-World Cases

- **API evolution**: Adding a new overload can change method resolution for existing calls, causing unexpected behavior.
- **Overloading with inheritance**: A subclass adding an overload can shadow the superclass's method for certain argument types.
- **Collections**: Methods like `remove(int)` vs `remove(Object)` in `List` can cause confusion due to static binding.

### References

- GitHub – Static vs Dynamic Binding - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/11-static-vs-dynamic-binding.md
- Baeldung – Static and Dynamic Binding in Java - https://www.baeldung.com/java-static-dynamic-binding
- SEI CERT – MET50-J: Avoid ambiguous or confusing uses of overloading - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1946-865/MET50-J.+Avoid+ambiguous+or+conf_da68044bf3914b329fa7d361b584e681-201124-1946-866.pdf


## Core Concept 4: Overloading Rules

### Definitions

**Core Definition**: Overloading rules are the specific conditions that must be met for two or more methods with the same name to be considered valid overloads by the Java compiler.

**Technical Definition**: Java supports overloading methods and can distinguish between methods with different method signatures. Methods within a class can have the same name if they have different parameter lists. The method to be invoked is determined at compile time based on the static types of the arguments. The return type alone is not sufficient to distinguish two overloads.

**Beginner-Friendly Explanation**: The rules of overloading are like the rules of a game. You can have two players with the same name, but they must wear different jerseys (different parameter lists). If they wear the same jersey number (same signature), the referee (compiler) won't allow it.

### Purposes

- To establish clear, deterministic rules for method resolution.
- To prevent ambiguous or confusing method declarations.
- To ensure that overloaded methods are distinguishable by the compiler.
- To guide developers in writing maintainable, understandable APIs.

### Syntax Rules and Structure

**The Rules of Method Overloading:**

| Rule | Description |
|------|-------------|
| **Rule 1: Same Name** | All overloaded methods must share the exact same name. |
| **Rule 2: Different Parameters** | The parameter list must differ in type, number, or order. |
| **Rule 3: Return Type Alone Insufficient** | Two methods differing only in return type are not valid overloads. |
| **Rule 4: Access Modifiers Can Differ** | Overloaded methods may have different access levels. |
| **Rule 5: Exceptions Can Differ** | Overloaded methods may throw different exceptions. |
| **Rule 6: Can Span Inheritance** | A subclass can add new overloads alongside inherited ones. |

**Complete General Syntax (Valid Overloads):**

```java
// Valid: different parameter types
void show(int x) { }
void show(double x) { }

// Valid: different number of parameters
void show(int x) { }
void show(int x, int y) { }

// Valid: different order of parameter types
void show(int x, String y) { }
void show(String x, int y) { }
```

**Complete General Syntax (Invalid Overloads):**

```java
// INVALID: return type alone differs
int show(int x) { return x; }
void show(int x) { } // Compile error: duplicate method

// INVALID: only parameter names differ
void show(int x) { }
void show(int y) { } // Compile error: same signature
```

**Syntax Rules:**

- The method name must stay exactly the same across all overloaded versions.
- The parameter list must differ; return type, access modifiers, and exceptions do not distinguish overloads.
- Two methods with identical parameter lists but different return types will not compile.
- Overloading can happen within the same class or between a superclass and subclass.

**Constraints and Limitations:**

- Ambiguous overloading (e.g., `test(ColoredPoint, Point)` vs `test(Point, ColoredPoint)`) causes compile-time errors.
- Methods with the same parameter types that differ only in declaration order are typically not flagged by compilers but can be error-prone.
- Overloading should be used sparingly; it can make code less readable.

### Annotated Complete Code Examples

**Example 1: Valid and Invalid Overloads**

```java
/**
 * Demonstrates overloading rules.
 */
public class OverloadRules {

    // Valid: different parameter type
    public void show(int x) {
        System.out.println("int: " + x);
    }

    // Valid: different number of parameters
    public void show(int x, int y) {
        System.out.println("two ints: " + x + ", " + y);
    }

    // Valid: different order of parameter types
    public void show(int x, String y) {
        System.out.println("int then String: " + x + ", " + y);
    }

    // INVALID: return type alone differs — this would not compile
    // public int show(int x) { return x; }

    // INVALID: only parameter name differs — this would not compile
    // public void show(int y) { }

    public static void main(String[] args) {
        OverloadRules rules = new OverloadRules();
        rules.show(5);
        rules.show(1, 2);
        rules.show(3, "Hello");
    }
}
```

**Expected Output:**

```
int: 5
two ints: 1, 2
int then String: 3, Hello
```

**Why This Output Occurs:**
- `show(int)` is valid by Rule 2 (different type from other overloads).
- `show(int, int)` is valid by Rule 2 (different number of parameters).
- `show(int, String)` is valid by Rule 2 (different order of parameter types).
- The commented-out methods would cause compile errors because they violate Rule 3 and the name-only rule.

### Real-World Cases

- **`println()`**: Follows all rules; overloaded by type.
- **`String.valueOf()`**: Overloaded for `int`, `long`, `float`, `double`, `boolean`, `char`, `Object`.
- **Anti-pattern example**: `getData(Integer)` returns formatted data; `getData(int)` returns raw data — confusing and violates the spirit of overloading.

### References

- GUVI – Method Overloading and Method Overriding in Java - https://www.guvi.in/hub/java-oops-concepts-tutorial/method-overloading-and-method-overriding/
- SEI CERT – MET50-J: Avoid ambiguous or confusing uses of overloading - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1946-865/MET50-J.+Avoid+ambiguous+or+conf_da68044bf3914b329fa7d361b584e681-201124-1946-866.pdf
- Brown University – Method Overloading - https://cs.brown.edu/courses/cs015/lecture/pdf/CS15.Lecture_15_Design_Patterns_and_Principles_Part_2.10.24.24.pdf


## Core Concept 5: Autoboxing and Varargs Ambiguities in Overloading Resolution

### Definitions

**Core Definition**: Autoboxing and varargs can introduce ambiguity or unexpected behavior in overload resolution because they add implicit conversions that the compiler must consider when selecting the most specific method.

**Technical Definition**: Autoboxing is the automatic conversion between primitive types and their corresponding wrapper classes (e.g., `int` ↔ `Integer`). Varargs allows a method to accept a variable number of arguments. When these features are combined with overloading, the compiler's three-phase resolution process (strict, loose, variable arity) can produce surprising results. For example, a method with `int` parameter is more specific than one with `Integer` when a primitive `int` argument is passed, but if only `Integer` exists, autoboxing kicks in. With varargs, ambiguity errors can arise when multiple overloads are applicable.

**Beginner-Friendly Explanation**: Autoboxing is like a translator who can turn "5" into a wrapped gift box. Varargs is like a bag that can hold any number of items. When you have multiple methods—some that take a box, some that take a bag—the compiler has to figure out which one you meant. Sometimes it gets confused and throws an "ambiguous method call" error.

### Purposes

- To understand how autoboxing and varargs affect overload resolution.
- To avoid writing ambiguous or confusing overloads involving these features.
- To know when to use `Integer` vs. `int` in overloaded methods.
- To diagnose and fix "reference to method is ambiguous" compile errors.

### Syntax Rules and Structure

**Three-Phase Overload Resolution:**

| Phase | Name | Allows |
|-------|------|--------|
| Phase 1 | Strict Invocation | No boxing/unboxing, no varargs |
| Phase 2 | Loose Invocation | Boxing/unboxing allowed, no varargs |
| Phase 3 | Variable Arity | Varargs, boxing, and unboxing allowed |

**Key Rules:**

- The compiler first searches for a match without boxing/unboxing or varargs.
- If none found, it allows boxing/unboxing but not varargs.
- If still none found, it allows varargs along with boxing/unboxing.
- Among applicable methods, the most specific (subtype) parameter types are chosen.

**Autoboxing Ambiguity Example:**

```java
// If only these two exist:
void log(Integer value) { }
void log(Object value) { }

log(1); // Autoboxes to Integer, calls log(Integer)
```

**Varargs Ambiguity Example:**

```java
void log(Object... args) { }
void log(int value, Object... args) { }

log(1, "message"); // Ambiguous! Both are applicable in Phase 3.
```

**Syntax Rules:**

- Prefer methods with primitive parameters when primitive arguments are expected.
- Avoid overloading varargs methods with other methods that could be applicable.
- Be as specific as possible when defining varargs methods to enforce strong type checking.
- Use non-ambiguous method signatures to avoid autoboxing pitfalls.

**Constraints and Limitations:**

- Autoboxing can silently change which overload is selected if a new overload is added.
- Varargs methods are treated as fixed-arity methods in Phase 1, which can change resolution.
- Ambiguity errors are compile-time errors; they cannot be resolved at runtime.
- Different compilers (e.g., Eclipse vs. javac) may behave differently in edge cases.

### Annotated Complete Code Examples

**Example 1: Autoboxing and Overload Selection**

```java
/**
 * Demonstrates how autoboxing affects overload selection.
 */
public class AutoboxingDemo {

    public void process(int value) {
        System.out.println("Primitive int: " + value);
    }

    public void process(Integer value) {
        System.out.println("Wrapper Integer: " + value);
    }

    public static void main(String[] args) {
        AutoboxingDemo demo = new AutoboxingDemo();

        // Primitive int matches process(int) first (Phase 1: strict invocation)
        demo.process(42);

        // Integer object matches process(Integer)
        Integer boxed = Integer.valueOf(42);
        demo.process(boxed);

        // If only process(Integer) existed, this would autobox to Integer
    }
}
```

**Expected Output:**

```
Primitive int: 42
Wrapper Integer: 42
```

**Why This Output Occurs:**
- `process(42)` passes a primitive `int`. Phase 1 (strict invocation) finds `process(int)` as an exact match, so it is selected over `process(Integer)`.
- `process(boxed)` passes an `Integer` object, which matches `process(Integer)` exactly.

**Example 2: Varargs Ambiguity**

```java
/**
 * Demonstrates varargs ambiguity.
 */
public class VarargsAmbiguity {

    // These two overloads are ambiguous when called with (1, "msg")
    public void log(Object... args) {
        System.out.println("Object varargs");
    }

    public void log(int value, Object... args) {
        System.out.println("int + Object varargs");
    }

    public static void main(String[] args) {
        VarargsAmbiguity demo = new VarargsAmbiguity();

        // Unambiguous: only log(Object...) is applicable
        demo.log("Hello");

        // AMBIGUOUS: both log(Object...) and log(int, Object...) are applicable
        // demo.log(1, "message"); // Compile error: reference to log is ambiguous
    }
}
```

**Expected Output:**

```
Object varargs
```

**Why This Output Occurs:**
- `log("Hello")` matches only `log(Object...)` because `int` is not involved.
- `log(1, "message")` is ambiguous: `log(Object...)` can accept `(Integer, String)` via autoboxing, and `log(int, Object...)` can accept `(int, String)`. Neither is more specific, causing a compile-time error.

**Workaround:**

```java
// Use explicit types to avoid ambiguity
public void log(int value, String message) {
    System.out.println("int + String: " + value + ", " + message);
}
```

### Real-World Cases

- **Logging frameworks**: Overloaded `log()` methods with `Object...` and specific types often cause ambiguity.
- **API design**: Adding an overload with a primitive type can change resolution for existing calls using wrapper types.
- **Collections**: `List.remove(int)` vs `List.remove(Object)` — autoboxing can cause unexpected method selection.
- **Generic methods**: Type erasure can create ambiguous signatures after compilation.

### References

- Stack Overflow – Why doesn't autoboxing overrule varargs when using method overloading in Java 7? - https://stackoverflow.com/questions/7689386/why-doesnt-autoboxing-overrule-varargs-when-using-method-overloading-in-java-7
- SEI CERT – MET50-J: Avoid ambiguous or confusing uses of overloading - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1946-865/MET50-J.+Avoid+ambiguous+or+conf_da68044bf3914b329fa7d361b584e681-201124-1946-866.pdf
- Oracle Help Center – Java Language Specification (Overload Resolution) - https://docs.oracle.com/javase/jp/26/docs/specs/jls/jls-15.html


## References

- Cornell Computer Science – Method Overloading - https://www.cs.cornell.edu/courses/cs100/1999su/P2%20Solutions/p2_5.html
- GUVI – Method Overloading and Method Overriding in Java - https://www.guvi.in/hub/java-oops-concepts-tutorial/method-overloading-and-method-overriding/
- SEI CERT – MET50-J: Avoid ambiguous or confusing uses of overloading - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1946-865/MET50-J.+Avoid+ambiguous+or+conf_da68044bf3914b329fa7d361b584e681-201124-1946-866.pdf
- GitHub – Static vs Dynamic Binding - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/11-static-vs-dynamic-binding.md
- Baeldung – Static and Dynamic Binding in Java - https://www.baeldung.com/java-static-dynamic-binding
- Brown University – Method Overloading - https://cs.brown.edu/courses/cs015/lecture/pdf/CS15.Lecture_15_Design_Patterns_and_Principles_Part_2.10.24.24.pdf
- Stack Overflow – Why doesn't autoboxing overrule varargs when using method overloading in Java 7? - https://stackoverflow.com/questions/7689386/why-doesnt-autoboxing-overrule-varargs-when-using-method-overloading-in-java-7
- Oracle Help Center – Java Language Specification (Overload Resolution) - https://docs.oracle.com/javase/jp/26/docs/specs/jls/jls-15.html
- IIT Bombay – Java Tutorial (Overloading) - https://www.cse.iitb.ac.in/~cs296/Java/JavaTut1_files/slide0056.htm
- Stack Overflow – Java overloading and overriding - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/2469767/java-overloading-and-overriding
- Oracle Help Center – Java Language Specification (Classes) - https://docs.oracle.com/javase/specs/jls/se16/html/jls-8.html
- Cornell Computer Science – Overloading vs. Overriding - https://www.cs.cornell.edu/courses/cs202/1999FA/lecture/f99-cs202-l1/tsld015.htm