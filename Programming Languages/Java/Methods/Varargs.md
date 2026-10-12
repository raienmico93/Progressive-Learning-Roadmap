# Java Variable Arguments (Varargs): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Variable arguments (varargs) is a Java language feature that allows a method or constructor to accept zero or more arguments of a specified type through a single parameter declaration.

**Technical Definition:** Varargs methods, formally known as variable arity methods (JLS §8.4.1), accept zero or more arguments of a specified type. The last formal parameter of a method or constructor is special: it may be a variable arity parameter, indicated by an ellipsis following the type. The compiler automatically wraps the arguments into an array of that type. The three periods after the final parameter's type indicate that the final argument may be passed as an array or as a sequence of arguments. Varargs can be used only in the final argument position.

**Beginner-Friendly Explanation:** Imagine you are ordering pizza for a group. Sometimes you order one pizza, sometimes five, sometimes none. Without varargs, you would need a separate method for each possible number of pizzas. With varargs, you can write one method that says "I can take any number of pizzas" — the method handles however many you give it. Varargs is Java's way of letting a method say "give me zero or more of these."

### Key Characteristics

- **Variable Arity:** A method can accept zero, one, or many arguments of the specified type.
- **Array Under the Hood:** The compiler automatically creates an array to hold the arguments passed to the varargs parameter.
- **Single Position:** Only one varargs parameter is permitted per method, and it must be the last parameter.
- **Upward Compatible:** The feature was designed to be compatible with pre-existing APIs that used array parameters.
- **Synergy with Autoboxing:** Varargs works seamlessly with autoboxing, allowing primitive values to be passed to `Object...` parameters.
- **Compile-Time Transformation:** The compiler translates the varargs parameter into an array parameter in the bytecode.

### Prerequisites

- Basic understanding of Java methods and parameter passing
- Familiarity with arrays (since varargs is compiled into an array)
- Knowledge of method overloading and method invocation
- Awareness of generics and type erasure (for understanding heap pollution)

### Related Programming Areas

- **API Design:** Varargs simplifies method signatures for APIs that accept a variable number of inputs (e.g., `String.format`, `printf`).
- **Reflection:** The `Method.invoke` and `Constructor.newInstance` methods use varargs for argument passing.
- **Message Formatting:** `MessageFormat.format` uses `Object... arguments` to accept a variable number of formatting arguments.
- **Generic Programming:** Varargs with generic types requires careful handling due to heap pollution risks.
- **Testing Frameworks:** Varargs enables flexible assertion methods (e.g., `assertEquals(expected, actual, message...)`).

### Core Concepts / Features

---

## 1. Varargs Syntax

### Definitions

**Core Definition:** The varargs syntax is the specific Java language construct used to declare a variable arity parameter using the ellipsis (`...`) following a type.

**Technical Definition:** In the Java Language Specification, the grammar for a variable arity parameter is:

```
VariableArityParameter:
    {VariableModifier} UnannType {Annotation} ... Identifier
```

This declares that the parameter is a variable arity parameter of the specified type.

**Beginner-Friendly Explanation:** The syntax is simple: put three dots (`...`) after the type of the last parameter. For example, `String...` means "any number of Strings." You write it just like a normal parameter but with the dots to signal that the method can take multiple values of that type.

### Purposes

- To declare a method parameter that accepts a variable number of arguments
- To provide a concise alternative to array parameters for callers who want to pass individual values
- To enable APIs that naturally work with a variable number of inputs

### Syntax Rules and Structure

**Complete General Syntax:**

```java
returnType methodName(fixedParamType fixedParam, varargsType... varargsParam) {
    // method body
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `returnType` | The method's return type. |
| `fixedParamType fixedParam` | Zero or more fixed parameters (optional). |
| `varargsType...` | The type of the varargs parameter, followed by three periods. |
| `varargsParam` | The parameter name; inside the method, it is an array of `varargsType`. |

**Syntax Rules:**

- The varargs parameter must be the last parameter in the method's parameter list.
- Only one varargs parameter is allowed per method.
- The varargs parameter is treated as an array inside the method body.
- Callers may pass zero or more arguments of the varargs type, or pass an array directly.

**Constraints and Limitations:**

- Varargs cannot be used with generic types without risking heap pollution (see Section 4).
- A method with a varargs parameter cannot also have an array parameter of the same type in a way that causes ambiguity.
- The ellipsis must be placed immediately after the type, not after the parameter name.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Varargs Method**

```java
// File: BasicVarargs.java
public class BasicVarargs {

    // Declares a varargs method: accepts zero or more String arguments
    public static void printAll(String... messages) {
        // Inside the method, 'messages' is a String[] array
        System.out.println("Number of arguments: " + messages.length);

        // Iterate over the array like any normal array
        for (String msg : messages) {
            System.out.println("  " + msg);
        }
    }

    public static void main(String[] args) {
        // Call with zero arguments
        printAll();
        // Output: Number of arguments: 0

        // Call with one argument
        printAll("Hello");
        // Output: Number of arguments: 1
        //           Hello

        // Call with multiple arguments
        printAll("Java", "is", "versatile");
        // Output: Number of arguments: 3
        //           Java
        //           is
        //           versatile

        // Call with an array (also valid)
        String[] arr = {"Array", "passed", "directly"};
        printAll(arr);
        // Output: Number of arguments: 3
        //           Array
        //           passed
        //           directly
    }
}
```

**Expected Output:**
```
Number of arguments: 0
Number of arguments: 1
  Hello
Number of arguments: 3
  Java
  is
  versatile
Number of arguments: 3
  Array
  passed
  directly
```

**Step-by-Step Explanation:**

1. `printAll(String... messages)` declares a varargs parameter. Inside the method, `messages` is a `String[]` array.
2. `printAll()` passes zero arguments; the compiler creates an empty `String[0]` array.
3. `printAll("Hello")` passes one argument; the compiler wraps it into a `String[]` of length 1.
4. `printAll("Java", "is", "versatile")` passes three arguments; the compiler creates a `String[]` of length 3.
5. `printAll(arr)` passes an existing array directly — no wrapping occurs.
6. In all cases, `messages.length` and the enhanced for-loop work because `messages` is a real array.

**Example 2: Varargs with Fixed Parameters**

```java
// File: VarargsWithFixed.java
public class VarargsWithFixed {

    // Fixed parameter first, then varargs — the standard pattern
    public static String format(String prefix, Object... values) {
        StringBuilder sb = new StringBuilder(prefix);
        for (Object value : values) {
            sb.append(" ").append(value);
        }
        return sb.toString();
    }

    public static void main(String[] args) {
        System.out.println(format("Log:", "User", "logged", "in"));
        // Output: Log: User logged in

        System.out.println(format("Result:", 42));
        // Output: Result: 42

        System.out.println(format("Empty:"));
        // Output: Empty:
    }
}
```

**Expected Output:**
```
Log: User logged in
Result: 42
Empty:
```

**Step-by-Step Explanation:**

1. `format(String prefix, Object... values)` has one fixed parameter (`prefix`) and one varargs parameter (`values`).
2. The varargs parameter must come last — this is enforced by the compiler.
3. In `main`, `format("Log:", "User", "logged", "in")` passes `"Log:"` to `prefix` and the remaining three strings to `values`.
4. `format("Empty:")` passes only the fixed argument, leaving `values` as an empty array.

### Real-World Cases

- **`String.format`:** `String.format(String format, Object... args)` — formats a string with a variable number of arguments.
- **`printf`:** `System.out.printf(String format, Object... args)` — prints formatted output.
- **`MessageFormat.format`:** `MessageFormat.format(String pattern, Object... arguments)` — formats messages with variable arguments.
- **`Method.invoke`:** `Method.invoke(Object obj, Object... args)` — invokes a method with a variable number of arguments.

---

## 2. Varargs Rules

### Definitions

**Core Definition:** Varargs rules are the mandatory constraints that govern how varargs parameters may be declared and used in Java.

**Technical Definition:** The JLS specifies that a method may have at most one variable arity parameter, and it must be the last formal parameter. The type of the variable arity parameter is an array type `T[]`, where `T` is the element type. The compiler generates an array creation at the call site when necessary.

**Beginner-Friendly Explanation:** The rules are simple: only one "any number of" parameter is allowed, and it has to be the last one on the list. Think of it like a checkout line — the "any number of items" is the last thing that goes through; you cannot have more items after it.

### Purposes

- To ensure the compiler can unambiguously determine which arguments belong to the varargs parameter
- To maintain compatibility with the array-based implementation
- To prevent ambiguous method signatures

### Syntax Rules and Structure

**Complete General Syntax (Rule Summary):**

```java
// VALID: varargs is the last parameter
public void method(int a, String... rest) { ... }

// VALID: varargs is the only parameter
public void method(String... args) { ... }

// INVALID: varargs is not the last parameter
public void method(String... args, int a) { ... }  // Compile error

// INVALID: two varargs parameters
public void method(String... args, int... nums) { ... }  // Compile error
```

**Component Breakdown:**

| Rule | Description |
|------|-------------|
| Last position | The varargs parameter must be the final parameter in the signature. |
| Single varargs | A method may declare only one varargs parameter. |
| Array type internally | The varargs parameter is treated as an array of its element type. |
| Zero or more | Callers may pass zero arguments, and the compiler creates an empty array. |

**Syntax Rules:**

- The varargs parameter must be the last parameter in the parameter list.
- Only one varargs parameter is permitted per method or constructor.
- The varargs parameter's type is the element type; inside the method, it is an array.
- The varargs parameter may be preceded by zero or more fixed parameters.

**Constraints and Limitations:**

- You cannot declare a varargs parameter in the middle of a parameter list.
- You cannot have two varargs parameters in the same method.
- The compiler creates a new array at each call site, which has performance implications.
- Overloading varargs methods is strongly discouraged because it creates ambiguity for callers.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Valid and Invalid Varargs Declarations**

```java
// File: VarargsRules.java
public class VarargsRules {

    // VALID: varargs is the last parameter
    public static void valid1(String label, int... numbers) {
        System.out.println(label + ": " + numbers.length + " numbers");
    }

    // VALID: varargs is the only parameter
    public static void valid2(double... values) {
        System.out.println("Received " + values.length + " doubles");
    }

    // INVALID: varargs is not last (uncomment to see compile error)
    // public static void invalid1(int... numbers, String label) { }

    // INVALID: two varargs parameters (uncomment to see compile error)
    // public static void invalid2(String... a, int... b) { }

    public static void main(String[] args) {
        valid1("Test", 1, 2, 3);        // Output: Test: 3 numbers
        valid1("Empty");                // Output: Empty: 0 numbers
        valid2(1.1, 2.2);               // Output: Received 2 doubles
        valid2();                       // Output: Received 0 doubles
    }
}
```

**Expected Output:**
```
Test: 3 numbers
Empty: 0 numbers
Received 2 doubles
Received 0 doubles
```

**Step-by-Step Explanation:**

1. `valid1(String label, int... numbers)` follows the rule: varargs is last.
2. `valid2(double... values)` follows the rule: only one varargs parameter.
3. The commented-out `invalid1` and `invalid2` methods would cause compile errors if uncommented because they violate the varargs placement and count rules.
4. `valid1("Empty")` demonstrates that zero varargs arguments are permitted — the compiler creates an empty `int[]`.

**Example 2: Passing an Array vs. Varargs Arguments**

```java
// File: ArrayVsVarargs.java
public class ArrayVsVarargs {

    public static int sum(int... numbers) {
        int total = 0;
        for (int n : numbers) total += n;
        return total;
    }

    public static void main(String[] args) {
        // Passing individual arguments (compiler wraps them)
        System.out.println("Sum1: " + sum(1, 2, 3, 4));
        // Output: Sum1: 10

        // Passing an array directly
        int[] arr = {5, 10, 15};
        System.out.println("Sum2: " + sum(arr));
        // Output: Sum2: 30

        // Passing zero arguments
        System.out.println("Sum3: " + sum());
        // Output: Sum3: 0
    }
}
```

**Expected Output:**
```
Sum1: 10
Sum2: 30
Sum3: 0
```

**Step-by-Step Explanation:**

1. `sum(1, 2, 3, 4)` passes four individual `int` arguments; the compiler creates an `int[]` containing them.
2. `sum(arr)` passes an existing array directly — the compiler does not create a new array in this case.
3. `sum()` passes zero arguments; the compiler creates an empty `int[]`, and the loop executes zero times, returning `0`.

### Real-World Cases

- **Logging Frameworks:** SLF4J's `Logger.info(String format, Object... arguments)` uses varargs for flexible logging.
- **Testing:** JUnit's `assertArrayEquals(T[] expected, T... actual)` uses varargs for comparing arrays.
- **Command-Line Tools:** Methods that process a variable number of input files or options.

---

## 3. Varargs Limitations

### Definitions

**Core Definition:** Varargs limitations are the constraints and potential pitfalls that arise from using varargs, including type safety issues, performance overhead, and ambiguity.

**Technical Definition:** The primary limitation is heap pollution: when a variable arity parameter has a non-reifiable type (e.g., `List<String>...`), the compiler generates an unchecked warning because the implicit array creation can lead to type-unsafe operations. A non-reifiable type is a type that is not completely available at runtime due to type erasure. Additionally, varargs methods can create ambiguity when overloaded, and the implicit array creation at each call site has a performance cost.

**Beginner-Friendly Explanation:** Varargs is convenient, but it comes with strings attached. If you use it with generic types like `List<String>...`, the compiler cannot fully check the types, which can lead to a crash at runtime. Also, every time you call a varargs method, Java creates a brand-new array behind the scenes — so calling it in a loop can waste memory.

### Purposes

- To inform developers of the risks and costs associated with varargs
- To guide safe usage patterns that avoid heap pollution
- To help developers decide when varargs is appropriate and when it is not

### Syntax Rules and Structure

**Key Limitations:**

| Limitation | Description |
|------------|-------------|
| Heap Pollution | Non-reifiable varargs types (e.g., `T...` where `T` is a type variable, or `List<String>...`) can cause `ClassCastException` at runtime. |
| Performance Overhead | A new array is allocated at each call site, which can be costly in performance-critical code. |
| Overloading Ambiguity | Overloading varargs methods creates confusion about which method is invoked. |
| No Fixed-Parameter After Varargs | Varargs must be last, so you cannot have required parameters after the variable ones. |
| Escape of Varargs Array | If the implicitly created array is exposed to callers, it can lead to type-safety violations. |

**Syntax Rules:**

- Using a non-reifiable type (e.g., `T...`, `List<String>...`) as a varargs parameter type generates a compile-time unchecked warning.
- The `@SafeVarargs` annotation can suppress the warning if the method body is proven safe.
- Overloading varargs methods is legal but strongly discouraged.

**Constraints and Limitations:**

- A method with a generic varargs parameter cannot be proven safe by the compiler in general.
- The `@SafeVarargs` annotation can only be applied to `static`, `final`, or `private` methods, or constructors (prior to Java 9, it was restricted to `static` and `final`).
- Varargs methods cannot be abstract if they have a non-reifiable parameter type (without warnings).

### Annotated Complete Step-by-Step Code Examples

**Example 1: Heap Pollution Demonstration**

```java
// File: HeapPollutionDemo.java
import java.util.*;

public class HeapPollutionDemo {

    // Non-reifiable varargs type: List<String>...
    // The compiler generates an unchecked warning here
    @SafeVarargs  // We claim the method is safe
    static void addToList(List<String>... lists) {
        // Inside, 'lists' is a List<String>[] array
        // But due to type erasure, the JVM sees it as List[]
        System.out.println("Number of lists: " + lists.length);
    }

    // UNSAFE method that causes heap pollution
    @SafeVarargs
    static String firstOfFirst(List<String>... lists) {
        List<Integer> ints = Collections.singletonList(42);
        Object[] objects = lists;  // This is allowed because of erasure
        objects[0] = ints;          // Heap pollution: storing List<Integer> in List<String>[]
        return lists[0].get(0);     // ClassCastException at runtime
    }

    public static void main(String[] args) {
        // This compiles but produces a warning if @SafeVarargs is absent
        addToList(Arrays.asList("a", "b"), Collections.emptyList());
        // Output: Number of lists: 2

        // This causes a ClassCastException at runtime
        try {
            String result = firstOfFirst(Arrays.asList("one", "two"));
            System.out.println(result);
        } catch (ClassCastException e) {
            System.out.println("ClassCastException caught: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Number of lists: 2
ClassCastException caught: class java.lang.Integer cannot be cast to class java.lang.String
```

**Step-by-Step Explanation:**

1. `addToList(List<String>... lists)` declares a varargs parameter with a generic type. Without `@SafeVarargs`, the compiler issues an unchecked warning.
2. `firstOfFirst` demonstrates heap pollution. The method receives `List<String>[]` but treats it as `Object[]`, storing a `List<Integer>` into it.
3. The `return lists[0].get(0)` line attempts to return an `Integer` as a `String`, causing a `ClassCastException`.
4. This occurs because type erasure removes the generic type information at runtime, so the JVM cannot prevent the type violation.

**Example 2: Performance Overhead**

```java
// File: VarargsPerformance.java
public class VarargsPerformance {

    // Varargs method: creates a new array at each call
    public static int sumVarargs(int... numbers) {
        int total = 0;
        for (int n : numbers) total += n;
        return total;
    }

    // Array parameter: no array creation at call site if array is reused
    public static int sumArray(int[] numbers) {
        int total = 0;
        for (int n : numbers) total += n;
        return total;
    }

    public static void main(String[] args) {
        // Each call creates a new array
        long start1 = System.nanoTime();
        for (int i = 0; i < 1_000_000; i++) {
            sumVarargs(1, 2, 3);
        }
        long end1 = System.nanoTime();
        System.out.println("Varargs time: " + (end1 - start1) / 1_000_000 + " ms");

        // Reuse a pre-created array
        int[] arr = {1, 2, 3};
        long start2 = System.nanoTime();
        for (int i = 0; i < 1_000_000; i++) {
            sumArray(arr);
        }
        long end2 = System.nanoTime();
        System.out.println("Array time:   " + (end2 - start2) / 1_000_000 + " ms");
    }
}
```

**Expected Output (approximate):**
```
Varargs time: 45 ms
Array time:   12 ms
```

**Step-by-Step Explanation:**

1. `sumVarargs(1, 2, 3)` creates a new `int[]` array at each call site, adding allocation overhead.
2. `sumArray(arr)` reuses the same array across all iterations — no allocation occurs inside the loop.
3. The timing difference demonstrates that varargs has a measurable performance cost when called frequently.
4. For performance-critical loops, consider providing an array-parameter overload.

### Real-World Cases

- **Generic APIs:** Methods like `Arrays.asList(T... a)` are safe because they only read from the array. Methods that store into the array are unsafe.
- **Frameworks:** Overloading varargs methods (e.g., `log(String... msgs)` and `log(String msg1, String msg2)`) can confuse callers.
- **High-Performance Code:** Avoiding varargs in tight loops to prevent unnecessary array allocations.

---

## 4. Varargs Best Practices

### Definitions

**Core Definition:** Varargs best practices are the recommended guidelines for using varargs safely, efficiently, and clearly in Java code.

**Technical Definition:** Best practices for varargs derive from authoritative sources including the SEI CERT Oracle Coding Standard and *Effective Java*. Key guidelines include: use varargs sparingly and only when the benefit is compelling; do not overload varargs methods; use `@SafeVarargs` only when the method is provably safe; and prefer explicit array parameters in performance-critical code.

**Beginner-Friendly Explanation:** Varargs is a powerful tool, but like any power tool, it can cause accidents if used carelessly. The best practices tell you: use it when it makes the API clearer, avoid overloading it, and never let the hidden array escape the method.

### Purposes

- To prevent heap pollution and runtime type errors
- To avoid confusion caused by overloading varargs methods
- To minimise performance overhead in critical code paths
- To maintain clear, unambiguous method signatures

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Use sparingly | Only use varargs when the benefit is truly compelling. |
| Avoid overloading varargs | Overloading creates ambiguity; use distinct method names instead. |
| Don't let the array escape | Never return the varargs array or expose it to callers. |
| Don't store into the array | Do not write to the implicitly created array (causes heap pollution). |
| Use `@SafeVarargs` carefully | Apply only to `static`, `final`, or `private` methods that are provably safe. |
| Prefer explicit arrays for performance | In hot loops, provide an array-parameter overload. |
| Use most specific type | Enable compile-time type checking by using the most specific type possible. |

**Syntax Rules:**

- The `@SafeVarargs` annotation may be applied only to methods or constructors that are `static`, `final`, or `private` (or constructors in Java 9+).
- `@SafeVarargs` suppresses unchecked warnings about non-reifiable varargs types.
- When overloading varargs methods, ensure the overloads are unambiguous (e.g., by using a fixed parameter before the varargs).

**Constraints and Limitations:**

- `@SafeVarargs` is a programmer assertion — it does not make the method safe; it only suppresses the warning.
- Overloading varargs methods is sometimes acceptable (e.g., `foo()`, `foo(int a1)`, `foo(int a1, int a2, int... rest)`) when the signatures are unambiguous.
- Varargs should not be retrofitted onto existing methods with array parameters without careful consideration.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Safe Varargs with @SafeVarargs**

```java
// File: SafeVarargsDemo.java
import java.util.*;

public class SafeVarargsDemo {

    // SAFE: Only reads from the varargs array; does not store into it
    @SafeVarargs
    public static <T> List<T> listOf(T... elements) {
        // Creates a new list from the elements — does not expose the array
        List<T> result = new ArrayList<>();
        for (T element : elements) {
            result.add(element);
        }
        return result;
    }

    public static void main(String[] args) {
        List<String> strings = listOf("a", "b", "c");
        System.out.println(strings);  // Output: [a, b, c]

        List<Integer> numbers = listOf(1, 2, 3);
        System.out.println(numbers);  // Output: [1, 2, 3]
    }
}
```

**Expected Output:**
```
[a, b, c]
[1, 2, 3]
```

**Step-by-Step Explanation:**

1. `listOf(T... elements)` is a generic varargs method. Without `@SafeVarargs`, the compiler would issue an unchecked warning.
2. The method only **reads** from the `elements` array — it does not store into it, and it does not let the array escape.
3. `@SafeVarargs` is applied to suppress the warning because the method is provably safe.
4. `listOf("a", "b", "c")` and `listOf(1, 2, 3)` demonstrate that the method works with different types.

**Example 2: Avoiding Overloading Varargs**

```java
// File: NoOverloadVarargs.java
public class NoOverloadVarargs {

    // COMPLIANT: Distinct method names avoid ambiguity
    public static void logMessage(String message) {
        System.out.println("[LOG] " + message);
    }

    public static void logMessages(String... messages) {
        for (String msg : messages) {
            System.out.println("[LOG] " + msg);
        }
    }

    public static void main(String[] args) {
        logMessage("Single message");
        // Output: [LOG] Single message

        logMessages("First", "Second", "Third");
        // Output: [LOG] First
        //         [LOG] Second
        //         [LOG] Third
    }
}
```

**Expected Output:**
```
[LOG] Single message
[LOG] First
[LOG] Second
[LOG] Third
```

**Step-by-Step Explanation:**

1. Instead of overloading a single `log` method with different parameter lists, two distinct method names are used: `logMessage` and `logMessages`.
2. This eliminates ambiguity — the caller knows exactly which method is being invoked.
3. The SEI CERT guideline DCL08-J recommends this approach to avoid confusion.

**Example 3: Performance-Optimised Varargs with Array Overload**

```java
// File: OptimisedVarargs.java
public class OptimisedVarargs {

    // Varargs version: convenient but allocates an array each call
    public static int sum(int... numbers) {
        int total = 0;
        for (int n : numbers) total += n;
        return total;
    }

    // Array version: no allocation if the caller reuses the array
    public static int sum(int[] numbers) {
        int total = 0;
        for (int n : numbers) total += n;
        return total;
    }

    public static void main(String[] args) {
        // Convenient call: compiler creates an array
        System.out.println("Varargs: " + sum(1, 2, 3));
        // Output: Varargs: 6

        // Performance-critical call: reuse a pre-created array
        int[] data = {10, 20, 30};
        int result = 0;
        for (int i = 0; i < 1000; i++) {
            result = sum(data);  // No array allocation
        }
        System.out.println("Array: " + result);
        // Output: Array: 60
    }
}
```

**Expected Output:**
```
Varargs: 6
Array: 60
```

**Step-by-Step Explanation:**

1. The `sum(int...)` varargs method is convenient for callers passing individual values.
2. The `sum(int[])` overload allows callers to pass a pre-existing array, avoiding allocation in performance-sensitive code.
3. In the loop, `sum(data)` calls the array overload directly — no new array is created.

### Real-World Cases

- **`Arrays.asList(T... a)`:** This method is safe because it only reads from the array to create a list. It is annotated with `@SafeVarargs` in the JDK.
- **`List.of(E... elements)`:** Java 9's `List.of` uses varargs safely to create immutable lists.
- **Logging Frameworks:** SLF4J uses varargs for parameterised logging, and it is carefully documented as safe.
- **Testing Frameworks:** JUnit's assertion methods use varargs for optional message parameters.

---

## References Links

- Varargs — Oracle Java Documentation - https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html
- Java Language Specification, §8.4.1: Formal Parameters - https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.1
- Improved Compiler Warnings and Errors When Using Non-Reifiable Formal Parameters with Varargs Methods — Oracle - https://docs.oracle.com/javase/8/docs/technotes/guides/language/non-reifiable-varargs.html
- SafeVarargs (Java SE 25 & JDK 25) — Oracle - https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/SafeVarargs.html
- Varargs in Java — Baeldung - https://www.baeldung.com/java-varargs
- DCL08-J. Avoid overloading varargs methods — SEI CERT Oracle Coding Standard for Java - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88886140
- DCL02-J. Use variable arguments (varargs) in moderation — SEI CERT Oracle Coding Standard for Java - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88886140
- Effective Java (3rd Edition) by Joshua Bloch — Item 53: Use varargs judiciously
- Heap Pollution — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/nonReifiableVarargsType.html
- Non-Reifiable Types — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/non-reifiable-varargs.html