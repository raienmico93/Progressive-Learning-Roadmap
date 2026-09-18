# Java Bounded Type Parameters: A Comprehensive Cheat Sheet

---

## Topic Overview

### Core Definition

**Bounded Type Parameters** are a feature of Java generics that restrict the types that can be used as type arguments in a parameterized type, ensuring that only specific types or their subtypes are accepted.

### Technical Definition

A bounded type parameter is declared by listing the type parameter's name, followed by the `extends` keyword, followed by its upper bound. In this context, `extends` is used in a general sense to mean either "extends" (as in classes) or "implements" (as in interfaces). A type variable with multiple bounds is a subtype of all the types listed in the bound; if one of the bounds is a class, it must be specified first.

### Beginner-Friendly Explanation

Imagine you are building a machine that processes documents. You could design it to accept *any* object, but then you would need to check what kind of object it is every time. Bounded type parameters let you say, "This machine only accepts documents that are printable and can be stapled." The compiler then ensures that only such documents are ever placed in the machine, so you never have to check at runtime.

### Key Characteristics

- **Type restriction**: Limits the types that can be used as type arguments.
- **Method invocation**: Allows invoking methods defined in the bound type (e.g., `doubleValue()` on `Number`).
- **Compile-time verification**: Enforces type constraints before the code runs.
- **Intersection types**: Multiple bounds create an intersection type that has all the members of the listed types.

### Prerequisites

- Basic understanding of Java generics (type parameters, generic classes, and methods).
- Familiarity with Java inheritance and interfaces.
- Knowledge of the `Comparable` and `Number` classes.

### Related Programming Areas

- **Generic Algorithms**: Sorting, searching, and mathematical operations.
- **Collections Framework**: `Collections.sort`, `Collections.max`.
- **API Design**: Creating flexible yet type-safe APIs.
- **Design Patterns**: Factory methods, builders, and strategy patterns.

### Core Concepts / Features

1. Upper Bounds (`extends`)
2. Multiple Bounds (`&`)

---

## Core Concept 1: Upper Bounds (`extends`)

### Definitions

**Core Definition**: An upper bound restricts the type parameter to a specific type or its subclasses.

**Technical Definition**: An upper-bounded type parameter is declared using the syntax `<T extends Bound>`, where `Bound` is the upper bound. The type argument used must be a subtype of `Bound`. If no bound is declared for a type variable, `Object` is assumed as the default upper bound.

**Beginner-Friendly Explanation**: An upper bound is like a ceiling. You can use any type as long as it is at or below that ceiling in the class hierarchy. For example, `<T extends Number>` means "T can be `Integer`, `Double`, `Float`, or any other subclass of `Number`, but not `String`."

### Purposes

- To restrict the types that can be used with a generic class or method.
- To allow invoking methods defined in the bound type (e.g., `intValue()` on `Number`).
- To enforce semantic constraints (e.g., only `Comparable` types for sorting).

### Syntax Rules and Structure

**Complete General Syntax:**

```java
<T extends UpperBound>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<T` | The type parameter. |
| `extends` | Keyword indicating an upper bound. |
| `UpperBound` | The class or interface that T must extend/implement. |

**Syntax Rules:**

- The `extends` keyword is used for both class inheritance and interface implementation in bounded type parameters.
- The bound can be a class, an interface, or another type variable.
- If no bound is specified, `Object` is assumed.

**Constraints and Limitations:**

- Cannot use primitive types as type arguments (e.g., `<T extends int>` is illegal).
- Cannot use `super` in type parameter bounds (only in wildcards).
- The bound type cannot be a final class if it is used as an upper bound.

### Annotated Complete Code Examples

**Example 1: Bounded Type Parameter with Number**

```java
/**
 * A generic class that only accepts Number subtypes.
 * @param <T> a type that extends Number
 */
public class NumberBox<T extends Number> {
    private T value;

    public NumberBox(T value) {
        this.value = value;
    }

    /**
     * Returns the value as a double.
     * Allowed because T extends Number, which has doubleValue().
     */
    public double asDouble() {
        return value.doubleValue();
    }

    public static void main(String[] args) {
        NumberBox<Integer> intBox = new NumberBox<>(10);
        System.out.println("Integer as double: " + intBox.asDouble());

        NumberBox<Double> doubleBox = new NumberBox<>(3.14);
        System.out.println("Double as double: " + doubleBox.asDouble());

        // NumberBox<String> stringBox = new NumberBox<>("hello"); // Compile error!
    }
}
```

**Expected Output:**

```
Integer as double: 10.0
Double as double: 3.14
```

**Why This Output Occurs:**
- `T extends Number` ensures that only `Number` subclasses (`Integer`, `Double`, etc.) can be used.
- This allows calling `value.doubleValue()`, a method defined in `Number`.
- Attempting to create `NumberBox<String>` causes a compile-time error because `String` does not extend `Number`.

**Step-by-Step Setup Guide:**
1. Create `NumberBox.java`.
2. Compile with `javac NumberBox.java`.
3. Run with `java NumberBox`.
4. Observe the output.

**Example 2: Bounded Type Parameter with Comparable**

```java
/**
 * A generic method that finds the maximum element in an array.
 */
public class ArrayUtils {

    /**
     * Returns the maximum element in a non-empty array.
     * @param array the array to search
     * @param <T> a type that extends Comparable
     * @return the maximum element
     */
    public static <T extends Comparable<T>> T max(T[] array) {
        if (array == null || array.length == 0) {
            throw new IllegalArgumentException("Array must not be empty");
        }
        T max = array[0];
        for (int i = 1; i < array.length; i++) {
            if (array[i].compareTo(max) > 0) {
                max = array[i];
            }
        }
        return max;
    }

    public static void main(String[] args) {
        Integer[] ints = {3, 1, 4, 1, 5, 9, 2, 6};
        System.out.println("Max int: " + max(ints));

        String[] strs = {"apple", "orange", "banana", "kiwi"};
        System.out.println("Max string: " + max(strs));

        Double[] doubles = {1.5, 3.7, 2.2, 0.8};
        System.out.println("Max double: " + max(doubles));
    }
}
```

**Expected Output:**

```
Max int: 9
Max string: orange
Max double: 3.7
```

**Why This Output Occurs:**
- The generic method `max` requires `T extends Comparable<T>`, ensuring that elements can be compared using `compareTo`.
- For `Integer[]`, the natural ordering yields `9` as the maximum.
- For `String[]`, lexicographic ordering yields `"orange"` (since `"o" > "k" > "b"` etc.).
- For `Double[]`, numeric ordering yields `3.7` as the maximum.
- Type inference determines `T` from the array type passed to the method.

### Real-World Cases

- **Sorting**: `Collections.sort(List<T>)` requires `T extends Comparable<T>` to compare elements.
- **Numeric Operations**: Methods that perform arithmetic on numbers use `<T extends Number>`.
- **Event Handling**: `<T extends Event>` for generic event listeners.
- **API Design**: `Optional<T>` and `Stream<T>` use bounded type parameters for utility methods.

### References

- Oracle Java Tutorials – Bounded Type Parameters - https://docs.oracle.com/javase/tutorial/java/generics/bounded.html
- Oracle Java Tutorials – Generic Methods and Bounded Type Parameters - https://docs.oracle.com/javase/tutorial/java/generics/boundedTypeParams.html
- Java Language Specification – Type Bounds - https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.4

---

## Core Concept 2: Multiple Bounds (`&`)

### Definitions

**Core Definition**: Multiple bounds restrict the type parameter to comply with multiple types, using the ampersand (`&`) to separate them.

**Technical Definition**: A type parameter with multiple bounds is a subtype of all the types listed in the bound. The syntax is `<T extends B1 & B2 & B3>`, where `B1`, `B2`, and `B3` are the bounds. If one of the bounds is a class, it must be specified first; the remaining bounds must be interfaces.

**Beginner-Friendly Explanation**: Multiple bounds are like requiring an employee to have several qualifications. You might say, "This person must be a manager *and* certified in first aid *and* fluent in Spanish." In Java, you can say, "This type must extend `ClassA` and implement `InterfaceB` and `InterfaceC`."

### Purposes

- To enforce that a type satisfies multiple interface contracts simultaneously.
- To combine class inheritance with interface implementation in a single type parameter.
- To enable invoking methods from all bound types within the generic code.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
<T extends B1 & B2 & B3>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<T` | The type parameter. |
| `extends` | Keyword indicating an upper bound. |
| `B1` | The first bound; if it is a class, it must be listed first. |
| `& B2 & B3` | Additional bounds; these must be interfaces. |

**Syntax Rules:**

- Only one class can be listed as a bound, and it must be the first type in the list.
- Multiple interfaces can be listed after the class (or after the first interface if no class is used).
- The order of types in a bound is only significant in that the erasure of a type variable is determined by the first type in its bound.
- A type variable may not be a subtype of two interface types which are different parameterizations of the same generic interface.

**Constraints and Limitations:**

- Cannot have multiple class bounds (only one class is allowed).
- The class bound, if present, must appear first.
- Cannot use `super` in type parameter bounds.
- The erasure of the type variable is determined by the leftmost bound.

### Annotated Complete Code Examples

**Example 1: Multiple Bounds with Class and Interfaces**

```java
import java.io.Serializable;

/**
 * A generic class with multiple bounds.
 * @param <T> a type that extends Number and implements Serializable
 */
public class MultiBoundBox<T extends Number & Serializable> {
    private T value;

    public MultiBoundBox(T value) {
        this.value = value;
    }

    public String describe() {
        return "Value: " + value + " (class: " + value.getClass().getSimpleName() + ")";
    }

    public static void main(String[] args) {
        MultiBoundBox<Integer> box = new MultiBoundBox<>(42);
        System.out.println(box.describe());

        // MultiBoundBox<String> box2 = new MultiBoundBox<>("hi"); // Compile error
    }
}
```

**Expected Output:**

```
Value: 42 (class: Integer)
```

**Why This Output Occurs:**
- `T extends Number & Serializable` requires `T` to be both a `Number` and `Serializable`.
- `Integer` satisfies both bounds, so it compiles.
- `String` does not extend `Number`, so it would cause a compile-time error.

**Example 2: Multiple Interface Bounds**

```java
interface CanFly {
    void fly();
}

interface CanRun {
    void run();
}

/**
 * A generic class that requires types implementing both CanFly and CanRun.
 */
class FlyAndRunHandler<T extends CanFly & CanRun> {
    void performAction(T animal) {
        animal.fly();
        animal.run();
    }
}

class Duck implements CanFly, CanRun {
    public void fly() {
        System.out.println("Duck is flying");
    }

    public void run() {
        System.out.println("Duck is running");
    }
}

public class MultipleBoundsDemo {
    public static void main(String[] args) {
        FlyAndRunHandler<Duck> handler = new FlyAndRunHandler<>();
        Duck duck = new Duck();
        handler.performAction(duck);
    }
}
```

**Expected Output:**

```
Duck is flying
Duck is running
```

**Why This Output Occurs:**
- `T extends CanFly & CanRun` requires that `T` implements both interfaces.
- `Duck` implements both `CanFly` and `CanRun`, so it satisfies the bound.
- The method `performAction` can call both `fly()` and `run()` because both methods are available through the bounds.
- If a class implemented only one of the interfaces, it would cause a compile-time error when used as a type argument.

### Real-World Cases

- **Factory Methods**: Creating objects that must be both a specific class and serializable.
- **Event Systems**: Types that must be both `Event` and `Comparable`.
- **Domain Models**: Entities that must extend a base class and implement auditing interfaces.
- **Generic Algorithms**: Methods that need to compare and serialize objects simultaneously.

### References

- Oracle Java Tutorials – Bounded Type Parameters (Multiple Bounds) - https://docs.oracle.com/javase/tutorial/java/generics/bounded.html
- Java Language Specification – Type Bounds - https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.4
- Java Language Specification – Additional Bound - https://docs.oracle.com/javase/specs/jls/se6/html/typesValues.html#3
- Dev.java – Bounded Type Parameters - https://dev.java/learn/generics/bounded-type-parameters/

---

## References

- Oracle Java Tutorials – Bounded Type Parameters - https://docs.oracle.com/javase/tutorial/java/generics/bounded.html
- Oracle Java Tutorials – Generic Methods and Bounded Type Parameters - https://docs.oracle.com/javase/tutorial/java/generics/boundedTypeParams.html
- Java Language Specification – Type Bounds - https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.4
- Java Language Specification – Additional Bound - https://docs.oracle.com/javase/specs/jls/se6/html/typesValues.html#3
- Dev.java – Bounded Type Parameters - https://dev.java/learn/generics/bounded-type-parameters/
- OpenJDK – Background: How We Got the Generics We Have - https://openjdk.org/projects/valhalla/design-notes/background