# Java Varargs: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Varargs** (variable-length arguments) is a Java language feature that allows a method to accept an arbitrary number of arguments of a specified type. It eliminates the need to manually create an array when invoking methods that take a variable number of values.

### Technical Definition

A variable arity (varargs) method is a method declared with a final formal parameter of the form `Type... name`. The compiler translates this declaration into an array parameter (`Type[] name`). At the call site, the compiler automatically boxes the individual arguments into an array (unless an array is passed directly). Varargs can only be used in the final parameter position, and a method can have at most one varargs parameter. When the varargs parameter uses a non-reifiable type (such as a generic type), the compiler issues an unchecked warning about potential **heap pollution**, which can be suppressed with `@SafeVarargs` under certain conditions .

### Beginner-Friendly Explanation

Imagine you're ordering coffee for your office. Some days it's just you (one coffee), some days it's your team (five coffees), and some days it's the whole company (fifty coffees). Instead of having different methods like `orderCoffeeFor1`, `orderCoffeeFor5`, and `orderCoffeeFor50`, varargs lets you write one method `orderCoffee(Coffee... cups)` that works for any number. Java automatically packages all the coffees into a box (array) for the barista.

### Key Characteristics

- **Arbitrary arity**: A varargs method can be called with zero or more arguments of the specified type.
- **Array backing**: Internally, the varargs parameter is treated as an array (`Type[]`).
- **Single per method**: Only one varargs parameter is allowed, and it must be the last parameter.
- **Autoboxing synergy**: Works seamlessly with autoboxing (e.g., `printf` with `Object...`).
- **Upward compatibility**: Existing methods taking arrays can be retrofitted with varargs without breaking callers.
- **Heap pollution risk**: Non-reifiable varargs types (like `List<String>...`) can cause unchecked warnings.
- **@SafeVarargs**: Suppresses warnings when the developer asserts the method is safe.

### Prerequisites

- Basic Java syntax (methods, arrays, parameters).
- Understanding of method overloading and parameter passing.
- Familiarity with generic types and type erasure (for the heap pollution section).

### Related Programming Areas

- **Core APIs**: `System.out.printf()`, `String.format()`, `MessageFormat.format()`.
- **Reflection**: `Method.invoke()`, `Constructor.newInstance()`.
- **Collections**: `List.of()`, `Set.of()`, `Arrays.asList()`.
- **Design Patterns**: Builder patterns, factory methods with optional parameters.

### Core Concepts / Features

1. Variable-Length Parameters
2. The `...` Syntax
3. Varargs Method Invocation (Arrays vs. Comma-Separated Lists)
4. Varargs Limitations (Last Parameter, Only One)
5. Performance Implications (Implicit Array Creation)
6. Heap Pollution Risks and `@SafeVarargs`


## Core Concept 1: Variable-Length Parameters

### Definitions

**Core Definition**: Variable-length parameters allow a method to accept a flexible number of arguments of a given type, from zero to many.

**Technical Definition**: A variable arity method is a method that can take a variable number of arguments. The method must contain at least one fixed argument, and the variable arity parameter is declared last. The compiler checks the types of all arguments, and all variable actual arguments must match the variable formal argument type. The presence of initial parameters of specific types is irrelevant; compile-time type checking remains effective for the fixed parameters .

**Beginner-Friendly Explanation**: Variable-length parameters are like a bag that can hold any number of items of the same kind. You can put in zero items, one item, or a hundred items—the bag doesn't care. It just holds them all and hands them to the method as a collection.

### Purposes

- To allow methods to accept an arbitrary number of arguments without requiring callers to create arrays.
- To simplify API usage for methods that naturally operate on a variable number of inputs.
- To maintain backward compatibility with existing array-based APIs.
- To reduce boilerplate code at call sites.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public ReturnType methodName(Type... parameterName) {
    // parameterName is treated as Type[]
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Type` | The type of the variable arguments (must be a reference type or primitive). |
| `...` | The ellipsis (three dots) indicating variable arity. |
| `parameterName` | The name of the parameter, treated as an array inside the method. |

**Syntax Rules:**

- The ellipsis follows the type and precedes the parameter name: `Type... name`.
- The varargs parameter must be the last parameter in the method's parameter list.
- Inside the method body, the varargs parameter is treated as an array of the declared type.
- The method can be called with zero arguments, individual arguments, or an array.

**Constraints and Limitations:**

- Only one varargs parameter is allowed per method.
- The varargs parameter must be in the final position.
- Using `Object` as the varargs type disables strong compile-time type checking.

### Annotated Complete Code Examples

**Example 1: A Simple Varargs Method**

```java
/**
 * Demonstrates a basic varargs method.
 */
public class VarargsBasics {

    // Varargs method: accepts any number of ints
    public static int sum(int... numbers) {
        int total = 0;
        for (int n : numbers) {  // numbers is treated as int[]
            total += n;
        }
        return total;
    }

    public static void main(String[] args) {
        // Zero arguments
        System.out.println("sum() = " + sum());

        // Three arguments
        System.out.println("sum(1, 2, 3) = " + sum(1, 2, 3));

        // Five arguments
        System.out.println("sum(10, 20, 30, 40, 50) = " + sum(10, 20, 30, 40, 50));

        // Passing an array directly
        int[] arr = {1, 2, 3, 4};
        System.out.println("sum(array) = " + sum(arr));
    }
}
```

**Expected Output:**

```
sum() = 0
sum(1, 2, 3) = 6
sum(10, 20, 30, 40, 50) = 150
sum(array) = 10
```

**Why This Output Occurs:**
- `sum()` passes an empty array, so the loop doesn't execute and returns `0`.
- `sum(1, 2, 3)` passes an array `{1, 2, 3}`, summing to `6`.
- `sum(10, 20, 30, 40, 50)` sums to `150`.
- `sum(arr)` passes the array directly, summing to `10`.

**Step-by-Step Setup Guide:**
1. Create `VarargsBasics.java`.
2. Compile with `javac VarargsBasics.java`.
3. Run with `java VarargsBasics`.
4. Observe the output.

### Real-World Cases

- **`printf()`**: `System.out.printf("%s: %d%n", name, id)` accepts a format string and any number of arguments.
- **`String.format()`**: `String.format("Hello, %s!", name)` for formatted output.
- **`List.of()`**: `List.of("a", "b", "c")` creates an immutable list from varargs.
- **`Arrays.asList()`**: `Arrays.asList(1, 2, 3)` converts varargs to a list.

### References

- Oracle Java Tutorials – Arbitrary Number of Arguments - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Oracle – Varargs (Java 8) - https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html
- SEI CERT – DCL58-J: Enable compile-time type checking of variable arity parameter types - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-2109-1027/DCL58-J.+Enable+compile-time+typ_78600b591dab401986583a75c60b7212-201124-2109-1028.pdf


## Core Concept 2: The `...` Syntax

### Definitions

**Core Definition**: The ellipsis (`...`) is the syntactic marker that declares a parameter as variable arity, following the type and preceding the parameter name.

**Technical Definition**: To use varargs, you follow the type of the last parameter by an ellipsis (three dots, `...`), then a space, and the parameter name. The three periods after the final parameter's type indicate that the final argument may be passed as an array or as a sequence of arguments. Varargs can be used only in the final argument position .

**Beginner-Friendly Explanation**: The `...` is like a label that says "this parameter can hold many of these." It's a signal to the compiler to pack up all the extra arguments into an array before passing them to the method.

### Purposes

- To clearly indicate to the compiler and other developers that a parameter is variable arity.
- To distinguish varargs parameters from regular array parameters.
- To enable the compiler to automatically package arguments into an array.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public ReturnType methodName(Type... parameterName)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Type` | The element type of the varargs parameter. |
| `...` | Three dots (ellipsis) indicating variable arity. |
| `parameterName` | The parameter name; inside the method, it is an array. |

**Syntax Rules:**

- The ellipsis must immediately follow the type (with optional whitespace).
- The parameter name follows the ellipsis.
- Inside the method body, the parameter is accessed as an array: `parameterName.length`, `parameterName[i]`.
- The varargs parameter can be passed as an array or as comma-separated values.

**Constraints and Limitations:**

- The ellipsis cannot be used with a parameter that is not last.
- Only one parameter can have the ellipsis.
- The ellipsis is not valid for return types or variable declarations.

### Annotated Complete Code Examples

**Example 1: The `...` Syntax in Practice**

```java
/**
 * Demonstrates the ... syntax for varargs.
 */
public class EllipsisSyntax {

    // Correct: ellipsis after type, before parameter name
    public static void printAll(String... messages) {
        for (String msg : messages) {
            System.out.println(msg);
        }
    }

    // Also correct: space between type and ellipsis
    public static void printNumbers(int ... numbers) {
        for (int n : numbers) {
            System.out.print(n + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        printAll("Hello", "World", "Varargs");
        printNumbers(1, 2, 3, 4, 5);
    }
}
```

**Expected Output:**

```
Hello
World
Varargs
1 2 3 4 5
```

**Why This Output Occurs:**
- `String... messages` declares a varargs parameter; inside the method, `messages` is a `String[]`.
- `int ... numbers` is also valid; the space between `int` and `...` is optional.
- The compiler packs the arguments into arrays automatically.

### Real-World Cases

- **`printf(String format, Object... args)`**: The `...` marks the variable arguments.
- **`String.format(String format, Object... args)`**: Same pattern.
- **`Method.invoke(Object obj, Object... args)`**: Reflection API uses varargs.

### References

- Oracle Java Tutorials – Arbitrary Number of Arguments - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Oracle – Varargs (Java 8) - https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html
- Java Language Specification – Formal Parameters - https://docs.oracle.com/javase/specs/jls/se16/html/jls-8.html#jls-8.4.1


## Core Concept 3: Varargs Method Invocation (Arrays vs. Comma-Separated Lists)

### Definitions

**Core Definition**: A varargs method can be invoked in two ways: by passing an explicit array, or by passing a comma-separated list of individual arguments.

**Technical Definition**: The method can be called either with an array or with a sequence of arguments. The code in the method body will treat the parameter as an array in either case . When invoked with individual arguments, the compiler creates an array to hold them. When invoked with an array, that array is passed directly without copying.

**Beginner-Friendly Explanation**: You can give the method a box of items you've already packed (an array), or you can hand it items one by one and let Java pack them for you (comma-separated list). Either way, the method opens the box and sees the same thing.

### Purposes

- To provide flexibility in how varargs methods are called.
- To maintain compatibility with existing APIs that take arrays.
- To simplify call sites when passing a known set of values.
- To allow dynamic argument lists when the number of arguments is determined at runtime.

### Syntax Rules and Structure

**Complete General Syntax (Comma-Separated):**

```java
methodName(arg1, arg2, arg3);
```

**Complete General Syntax (Array):**

```java
methodName(new Type[] {arg1, arg2, arg3});
```

**Component Breakdown:**

| Invocation Style | Description |
|------------------|-------------|
| Comma-separated | Compiler creates an array from the individual arguments. |
| Array | The array is passed directly; no new array is created. |

**Syntax Rules:**

- Both invocation styles are valid for varargs methods.
- When passing an array, the array's type must match the varargs element type.
- When passing individual arguments, each must be assignable to the varargs element type.
- An array can be passed directly, or a single element can be passed (which the compiler wraps in a one-element array).

**Constraints and Limitations:**

- Passing an array directly avoids the overhead of array creation.
- Passing individual arguments triggers implicit array creation.
- Mixing arrays and individual arguments is not allowed.

### Annotated Complete Code Examples

**Example 1: Two Ways to Invoke a Varargs Method**

```java
/**
 * Demonstrates array vs. comma-separated invocation of varargs.
 */
public class InvocationStyles {

    public static void display(String... items) {
        System.out.println("Count: " + items.length);
        for (String item : items) {
            System.out.println("  - " + item);
        }
    }

    public static void main(String[] args) {
        // Style 1: Comma-separated arguments (compiler creates array)
        System.out.println("Comma-separated:");
        display("apple", "banana", "cherry");

        // Style 2: Explicit array (passed directly)
        System.out.println("\nExplicit array:");
        String[] fruits = {"apple", "banana", "cherry"};
        display(fruits);

        // Style 3: Single element (wrapped in array)
        System.out.println("\nSingle element:");
        display("only-one");

        // Style 4: No arguments (empty array)
        System.out.println("\nNo arguments:");
        display();
    }
}
```

**Expected Output:**

```
Comma-separated:
Count: 3
  - apple
  - banana
  - cherry

Explicit array:
Count: 3
  - apple
  - banana
  - cherry

Single element:
Count: 1
  - only-one

No arguments:
Count: 0
```

**Why This Output Occurs:**
- `display("apple", "banana", "cherry")` creates a `String[]` with three elements.
- `display(fruits)` passes the `fruits` array directly; no new array is created.
- `display("only-one")` creates a one-element array.
- `display()` creates an empty array.

### Real-World Cases

- **`String.format()`**: Can be called with individual arguments or an `Object[]`.
- **`MessageFormat.format()`**: Accepts either an `Object[]` or varargs (in newer APIs).
- **Reflection**: `method.invoke(obj, arg1, arg2)` or `method.invoke(obj, argsArray)`.

### References

- Oracle Java Tutorials – Arbitrary Number of Arguments - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Oracle – Varargs (Java 8) - https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html
- Stack Overflow – How varargs work - https://stackoverflow.com/questions/12462079/possible-heap-pollution-via-varargs-parameter


## Core Concept 4: Varargs Limitations (Last Parameter, Only One)

### Definitions

**Core Definition**: Varargs has two strict syntactic limitations: the varargs parameter must be the **last parameter** in the method's parameter list, and there can be **only one** varargs parameter per method.

**Technical Definition**: Varargs can be used only in the final argument position. The method must contain at least one fixed argument, and the variable arity parameter is declared last . Because the compiler translates the varargs parameter into an array, having it anywhere but last would make the method signature ambiguous and the argument parsing impossible.

**Beginner-Friendly Explanation**: Think of a varargs parameter as a "catch-all" at the end of a line. If you put it in the middle, the compiler wouldn't know where the catch-all ends and the next parameter begins. Similarly, two catch-alls would compete for arguments, creating ambiguity.

### Purposes

- To ensure unambiguous parsing of arguments at the call site.
- To allow fixed parameters before the varargs parameter for context.
- To maintain a clear, predictable method signature.

### Syntax Rules and Structure

**Valid Syntax:**

```java
// Fixed parameters before varargs
public void method(String prefix, int count, Object... items) { }

// Only varargs
public void method(Object... items) { }
```

**Invalid Syntax:**

```java
// ERROR: varargs not last
public void method(Object... items, String suffix) { }

// ERROR: two varargs parameters
public void method(Object... items, int... numbers) { }
```

**Syntax Rules:**

- The varargs parameter must be the final parameter.
- Any number of fixed parameters may precede the varargs parameter.
- Only one varargs parameter is allowed per method.
- A method can have either a varargs parameter or an array parameter as the last parameter, but not both.

**Constraints and Limitations:**

- The compiler reports a compile-time error if these rules are violated.
- Overloading varargs methods can create ambiguity; use sparingly .

### Annotated Complete Code Examples

**Example 1: Valid Fixed Parameters Before Varargs**

```java
/**
 * Demonstrates fixed parameters preceding a varargs parameter.
 */
public class FixedBeforeVarargs {

    public static void log(String level, String... messages) {
        for (String msg : messages) {
            System.out.println("[" + level + "] " + msg);
        }
    }

    public static void main(String[] args) {
        log("INFO", "Application started", "Port: 8080");
        log("ERROR");
    }
}
```

**Expected Output:**

```
[INFO] Application started
[INFO] Port: 8080
[ERROR]
```

**Why This Output Occurs:**
- `level` is a fixed parameter consumed first.
- `messages` captures all remaining arguments as a `String[]`.
- `log("ERROR")` passes an empty array for `messages`, so no messages print.

### Real-World Cases

- **`printf(String format, Object... args)`**: Format string is fixed; arguments are varargs.
- **`String.format(Locale l, String format, Object... args)`**: Locale and format are fixed.
- **`MessageFormat.format(String pattern, Object... arguments)`**: Pattern is fixed.

### References

- Oracle – Varargs (Java 8) - https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html
- Oracle Java Tutorials – Arbitrary Number of Arguments - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Stack Overflow – Varargs limitations - https://stackoverflow.com/questions/12462079/possible-heap-pollution-via-varargs-parameter


## Core Concept 5: Performance Implications (Implicit Array Creation)

### Definitions

**Core Definition**: When a varargs method is invoked with individual arguments, the compiler creates an array to hold them. This **implicit array creation** has performance implications, especially in hot loops or performance-sensitive code.

**Technical Definition**: The compiler translates the varargs formal parameter into an array parameter. When the method is invoked with a sequence of arguments, the compiler creates a new array on the heap, copies the arguments into it, and passes it to the method. This allocation and copying add overhead compared to passing a pre-existing array. The JIT compiler may optimize away this array creation through escape analysis in some cases, but this is not guaranteed .

**Beginner-Friendly Explanation**: Every time you call a varargs method with individual arguments, Java has to make a new box, put your items in it, and hand you the box. If you're calling the method a million times in a loop, you're making a million boxes—that takes time and memory. If you already have a box (an array), you can just hand it over, no new box needed.

### Purposes

- To understand the hidden cost of varargs in performance-critical code.
- To know when to pass an array instead of individual arguments.
- To make informed decisions about varargs usage in hot paths.

### Syntax Rules and Structure

**Performance Considerations:**

| Scenario | Array Creation | Notes |
|----------|---------------|-------|
| Individual arguments | Yes | New array created each call |
| Pre-existing array | No | Array passed directly |
| Zero arguments | Yes (empty array) | May be cached by JIT |
| Single argument | Yes | One-element array |

**Syntax Rules:**

- Passing an array directly avoids implicit array creation.
- The JIT may optimize away array creation in some cases via escape analysis.
- Varargs methods in hot loops are candidates for refactoring to accept arrays directly.

**Constraints and Limitations:**

- Escape analysis optimization is not guaranteed and varies by JVM.
- Array creation adds garbage collection pressure.
- The overhead is typically negligible for non-hot paths.

### Annotated Complete Code Examples

**Example 1: Demonstrating Array Creation Overhead**

```java
/**
 * Demonstrates the array creation overhead of varargs.
 * Note: Actual timing may vary; this is illustrative.
 */
public class VarargsPerformance {

    // Varargs method: implicit array creation on each call
    public static int sumVarargs(int... numbers) {
        int total = 0;
        for (int n : numbers) {
            total += n;
        }
        return total;
    }

    // Array method: no implicit array creation
    public static int sumArray(int[] numbers) {
        int total = 0;
        for (int n : numbers) {
            total += n;
        }
        return total;
    }

    public static void main(String[] args) {
        // Hot loop with varargs (creates a new array each call)
        long start = System.nanoTime();
        for (int i = 0; i < 1_000_000; i++) {
            sumVarargs(1, 2, 3);
        }
        long varargsTime = System.nanoTime() - start;

        // Hot loop with array (creates the array once)
        int[] arr = {1, 2, 3};
        start = System.nanoTime();
        for (int i = 0; i < 1_000_000; i++) {
            sumArray(arr);
        }
        long arrayTime = System.nanoTime() - start;

        System.out.printf("Varargs: %d ms%n", varargsTime / 1_000_000);
        System.out.printf("Array:   %d ms%n", arrayTime / 1_000_000);
    }
}
```

**Expected Output (approximate, varies by JVM):**

```
Varargs: 15 ms
Array:   8 ms
```

**Why This Output Occurs:**
- `sumVarargs(1, 2, 3)` creates a new `int[]` on each call, adding allocation overhead.
- `sumArray(arr)` passes the pre-existing array, avoiding allocation.
- The JIT may optimize the varargs version, but the difference is often measurable.

**Step-by-Step Setup Guide:**
1. Create `VarargsPerformance.java`.
2. Compile with `javac VarargsPerformance.java`.
3. Run with `java VarargsPerformance`.
4. Observe the timing difference (results will vary).

### Real-World Cases

- **Hot loops**: Avoid varargs methods in performance-critical loops; pass arrays.
- **Logging**: `logger.debug("Value: {}", value)` may create an array; consider alternative APIs.
- **Math libraries**: Prefer array-accepting methods for numerical computation.

### References

- Stack Overflow – Varargs performance and escape analysis - https://stackoverflow.com/questions/12462079/possible-heap-pollution-via-varargs-parameter
- Oracle Java Tutorials – Arbitrary Number of Arguments - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Oracle – Varargs (Java 8) - https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html


## Core Concept 6: Heap Pollution Risks and `@SafeVarargs`

### Definitions

**Core Definition**: **Heap pollution** occurs when a variable of a parameterized type refers to an object that is not of that parameterized type. **`@SafeVarargs`** is an annotation that suppresses compiler warnings about potential heap pollution from non-reifiable varargs types.

**Technical Definition**: Heap pollution occurs when a variable of a parameterized type refers to an object that is not of that parameterized type. This situation occurs if the program performed some operation that gives rise to an unchecked warning at compile-time . When a varargs method uses a non-reifiable type (e.g., `List<String>...`), the compiler translates the varargs parameter to an array of the erased type (e.g., `List[]`). Because arrays are covariant and reified, it becomes possible to store an object of the wrong type into the array, leading to a `ClassCastException` at a later point . The `@SafeVarargs` annotation is a programmer assertion that the method does not perform potentially unsafe operations on its varargs parameter. It suppresses the unchecked warning at the declaration site .

**Beginner-Friendly Explanation**: Heap pollution is like a box labeled "Apples" that actually contains an Orange. If someone trusts the label and tries to eat an apple from the box, they'll be surprised to find an orange—and in Java, that's a `ClassCastException`. Varargs with generic types can create this situation because the compiler erases the generic type information. `@SafeVarargs` is your way of saying, "I promise this method is safe—I've checked it, and it won't put an orange in the apple box."

### Purposes

- To understand the risks of using varargs with generic (non-reifiable) types.
- To use `@SafeVarargs` to suppress warnings when the method is provably safe.
- To avoid runtime `ClassCastException` caused by heap pollution.
- To write type-safe generic varargs methods.

### Syntax Rules and Structure

**Complete General Syntax (Varargs with Generic Type):**

```java
// Compiler warning: Possible heap pollution
public static <T> void addToList(List<T> list, T... elements) {
    for (T element : elements) {
        list.add(element);
    }
}

// With @SafeVarargs
@SafeVarargs
public static <T> void addToList(List<T> list, T... elements) {
    for (T element : elements) {
        list.add(element);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@SafeVarargs` | Annotation asserting the method is safe. |
| `T...` | Non-reifiable varargs type (erased to `Object[]`). |

**`@SafeVarargs` Rules:**

- Applicable to static methods, final methods, constructors (Java 7+), and private instance methods (Java 9+) .
- The method must not perform potentially unsafe operations on the varargs array.
- Suppresses the "Possible heap pollution" unchecked warning.
- Does **not** suppress the "varargs" warning on call sites; use `@SuppressWarnings("varargs")` for that .

**Constraints and Limitations:**

- `@SafeVarargs` does not prevent heap pollution; it only suppresses the warning.
- If the method is not actually safe, users will not be warned, and runtime errors may occur .
- On interfaces, use `@SuppressWarnings("unchecked")` instead.
- The method must not cast the varargs array to `Object[]` or perform other unsafe operations.

### Annotated Complete Code Examples

**Example 1: Heap Pollution in Action**

```java
import java.util.*;

/**
 * Demonstrates heap pollution with varargs and generics.
 */
public class HeapPollutionDemo {

    // This method is NOT safe: it casts the varargs array to Object[]
    @SafeVarargs // Lying! This is not actually safe.
    public static void unsafeMethod(List<String>... lists) {
        Object[] array = lists; // Upcast to Object[]
        List<Integer> intList = Arrays.asList(42);
        array[0] = intList; // Stores Integer list in String list array!
    }

    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("hello");

        try {
            unsafeMethod(list);
            String s = list.get(0); // ClassCastException!
            System.out.println("String: " + s);
        } catch (ClassCastException e) {
            System.out.println("ClassCastException: Heap pollution detected!");
        }
    }
}
```

**Expected Output:**

```
ClassCastException: Heap pollution detected!
```

**Why This Output Occurs:**
- `unsafeMethod(List<String>... lists)` erases to `unsafeMethod(List[] lists)`.
- `Object[] array = lists` upcasts the `List[]` to `Object[]`.
- `array[0] = intList` stores a `List<Integer>` into what was declared as a `List<String>[]`.
- `list.get(0)` tries to cast the `Integer` to `String`, causing `ClassCastException`.

**Example 2: Safe Varargs Method**

```java
import java.util.*;

/**
 * Demonstrates a safe varargs method with @SafeVarargs.
 */
public class SafeVarargsDemo {

    @SafeVarargs
    public static <T> List<T> listOf(T... elements) {
        List<T> list = new ArrayList<>();
        for (T element : elements) {
            list.add(element);
        }
        return list;
    }

    public static void main(String[] args) {
        List<String> strings = listOf("a", "b", "c");
        System.out.println("Strings: " + strings);

        List<Integer> integers = listOf(1, 2, 3);
        System.out.println("Integers: " + integers);
    }
}
```

**Expected Output:**

```
Strings: [a, b, c]
Integers: [1, 2, 3]
```

**Why This Output Occurs:**
- `listOf` is annotated with `@SafeVarargs` because it only reads from the varargs array and adds elements to a new list.
- It does not cast the varargs array to `Object[]` or store elements of the wrong type.
- The compiler suppresses the heap pollution warning for this method.

### Real-World Cases

- **`List.of(E... elements)`**: Uses `@SafeVarargs` because it safely copies elements.
- **`Collections.addAll(Collection<? super T>, T... elements)`**: Safe because it adds elements to the collection.
- **`Arrays.asList(T... a)`**: Returns a list backed by the array; safe if the array is not modified.

### References

- Oracle – Non-Reifiable Types and Heap Pollution - https://docs.oracle.com/javase/tutorial/java/generics/nonReifiableVarargsType.html
- Oracle – @SafeVarargs Annotation (Java 24) - https://docs.oracle.com/javase/jp/24/docs/api/java.base/java/lang/SafeVarargs.html
- OpenJDK – @SafeVarargs vs @SuppressWarnings("varargs") - https://mail.openjdk.org/pipermail/compiler-dev/2020-May/014521.html
- Stack Overflow – Possible heap pollution via varargs parameter - https://stackoverflow.com/questions/12462079/possible-heap-pollution-via-varargs-parameter
- Java 9 @SafeVarargs Enhancement - https://github.com/Guru-1205/Java-DSA-OOPS-PrepHub/blob/main/JAVA%20ALL%20NOTES%20-%202.pdf


## References

- Oracle Java Tutorials – Arbitrary Number of Arguments - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Oracle – Varargs (Java 8) - https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html
- Oracle – Non-Reifiable Types and Heap Pollution - https://docs.oracle.com/javase/tutorial/java/generics/nonReifiableVarargsType.html
- Oracle – @SafeVarargs Annotation (Java 24) - https://docs.oracle.com/javase/jp/24/docs/api/java.base/java/lang/SafeVarargs.html
- SEI CERT – DCL58-J: Enable compile-time type checking of variable arity parameter types - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-2109-1027/DCL58-J.+Enable+compile-time+typ_78600b591dab401986583a75c60b7212-201124-2109-1028.pdf
- OpenJDK – @SafeVarargs vs @SuppressWarnings("varargs") - https://mail.openjdk.org/pipermail/compiler-dev/2020-May/014521.html
- Stack Overflow – Possible heap pollution via varargs parameter - https://stackoverflow.com/questions/12462079/possible-heap-pollution-via-varargs-parameter
- Java Language Specification – Formal Parameters - https://docs.oracle.com/javase/specs/jls/se16/html/jls-8.html#jls-8.4.1
- Java Language Specification – Method Invocation - https://docs.oracle.com/javase/specs/jls/se10/html/jls-15.html#jls-15.12.2.1