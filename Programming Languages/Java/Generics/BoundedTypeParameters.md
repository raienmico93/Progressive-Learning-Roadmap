# Java Bounded Type Parameters: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Bounded type parameters are type parameters that restrict the types that can be used as type arguments by specifying one or more bounds — a type must be a subtype of the specified bound to be used.

**Technical Definition:** Per the Java Language Specification, a type parameter is bounded if it declares one or more bounds. The syntax `<T extends B1 & B2 & B3>` declares a type variable `T` that is a subtype of all types listed in the bound. If one of the bounds is a class, it must be specified first. In this context, `extends` is used in a general sense to mean either "extends" (as in classes) or "implements" (as in interfaces). Unlike wildcards, bounded type parameters are used when you need to call methods defined in the bound — for example, `<T extends Number>` allows calling `doubleValue()` on values of type `T`.

**Beginner-Friendly Explanation:** A bounded type parameter is like saying "you can use any type, as long as it's a certain kind." For example, `<T extends Number>` means T can be `Integer`, `Double`, `Long`, etc., but not `String`. This is useful because it lets you call methods that all numbers have (like `doubleValue()`) inside your generic code. Think of it as a bouncer at a club who only lets in people who have a certain membership card — the bound defines what kind of card is required.

### Key Characteristics

- **Upper Bound Restriction:** The type argument must be the specified bound type or a subtype of it; `extends` is used for both classes and interfaces.
- **Multiple Bounds:** A type parameter can have multiple upper bounds separated by `&`, with at most one class bound, which must appear first.
- **Method Access:** Bounded type parameters allow calling methods defined in the bound, unlike unbounded type parameters which only allow `Object` methods.
- **Type Erasure Replacement:** The type parameter is erased to its first bound (or `Object` if unbounded) during compilation.
- **Recursive Bounds:** A type parameter can reference itself in its bound, enabling the Curiously Recurring Template Pattern (CRTP).
- **Lower Bounds Only in Wildcards:** Lower bounds (`? super T`) are only available with wildcards, not with type parameters in declarations.

### Prerequisites

- Basic understanding of Java generics (type parameters, generic classes, generic methods)
- Familiarity with Java inheritance and interfaces
- Knowledge of the Java Collections Framework
- Awareness of type erasure
- Java 5 or later

### Related Programming Areas

- **Generic Methods:** Bounded type parameters are commonly used in generic methods for sorting, searching, and comparing.
- **Collections Framework:** Methods like `Collections.max()` and `Collections.sort()` use bounded type parameters.
- **Enum Types:** The `Enum<E extends Enum<E>>` declaration is a classic example of recursive bounded type parameters.
- **Design Patterns:** Bounded type parameters enable type-safe builders and the Curiously Recurring Template Pattern.
- **Functional Programming:** Generic functional interfaces often use bounded type parameters (e.g., `<T extends Comparable<T>>`).

---

## Core Concepts / Features

---

## 1. Upper Bounds

### Definitions

**Core Definition:** An upper bound restricts a type parameter to be the specified type or a subtype of it, using the `extends` keyword.

**Technical Definition:** To declare a bounded type parameter, list the type parameter's name, followed by the `extends` keyword, followed by its upper bound. Note that, in this context, `extends` is used in a general sense to mean either "extends" (as in classes) or "implements" (as in interfaces). An upper bounded wildcard restricts the unknown type to be a specific type or a subtype of that type. It uses the syntax `<? extends Type>`.

**Beginner-Friendly Explanation:** An upper bound sets a ceiling — the type must be the specified type or "below" it in the hierarchy. For example, `<T extends Number>` means T can be `Integer`, `Double`, `Long`, etc., but not `String` or `Object`.

### Purposes

- To restrict type arguments to a type and its subtypes
- To enable calling methods defined in the bound type
- To provide type safety while allowing flexibility
- To enable reading (producing) from generic structures

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Bounded type parameter
<T extends UpperBound>

// Bounded wildcard
<? extends UpperBound>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `T` | The type parameter name |
| `extends` | The keyword indicating an upper bound (means "extends or implements") |
| `UpperBound` | The type that T must be a subtype of |

**Syntax Rules:**

- `extends` is used for both class and interface bounds.
- Only one class can be used as a bound, but any number of interfaces are allowed.
- The class bound (if present) must appear first.
- An upper bounded wildcard (`? extends T`) allows reading elements as type T but prevents writing (except `null`).

**Constraints and Limitations:**

- Cannot instantiate type parameters directly (`new T()` is illegal).
- Cannot create arrays of parameterized types (`new List<String>[10]` is illegal).
- For upper bounded wildcards, you cannot add elements (except `null`) because the exact type is unknown.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Upper Bound with Number**

```java
// File: UpperBoundDemo.java
import java.util.*;

public class UpperBoundDemo {

    // Bounded type parameter: T must be a Number
    // This allows calling doubleValue() on elements of type T
    public static <T extends Number> double sum(List<T> numbers) {
        double total = 0;
        for (T element : numbers) {
            total += element.doubleValue(); // Can call Number methods
        }
        return total;
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3, 4, 5);
        List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);

        System.out.println("Sum ints: " + sum(ints));       // 15.0
        System.out.println("Sum doubles: " + sum(doubles)); // 6.6

        // The following would be a compile-time error:
        // List<String> strings = Arrays.asList("A", "B");
        // sum(strings); // String is not a Number
    }
}
```

**Expected Output:**
```
Sum ints: 15.0
Sum doubles: 6.6
```

**Step-by-Step Explanation:**

1. `sum` declares `<T extends Number>`, restricting T to `Number` or its subtypes.
2. Inside the method, `element.doubleValue()` is legal because all `Number` objects have this method.
3. `sum(ints)` works because `Integer` extends `Number`.
4. `sum(doubles)` works because `Double` extends `Number`.
5. If `String` were used, the compiler would reject it because `String` does not extend `Number`.

**Example 2: Upper Bounded Wildcard**

```java
// File: UpperBoundedWildcardDemo.java
import java.util.*;

public class UpperBoundedWildcardDemo {

    // Upper bounded wildcard: reads numbers
    public static double sumOfList(List<? extends Number> list) {
        double sum = 0;
        for (Number n : list) {
            sum += n.doubleValue(); // Can read as Number
        }
        return sum;
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3);
        List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);

        System.out.println("Sum ints: " + sumOfList(ints));     // 6.0
        System.out.println("Sum doubles: " + sumOfList(doubles)); // 6.6
    }
}
```

**Expected Output:**
```
Sum ints: 6.0
Sum doubles: 6.6
```

### Real-World Cases

- **`<T extends Comparable<T>>`:** Used in sorting and comparison methods
- **`<T extends Number>`:** Used in numeric algorithms
- **`<T extends CharSequence>`:** Used in text processing
- **`List<? extends Number>`:** Accepts lists of any numeric type

---

## 2. Lower Bounds

### Definitions

**Core Definition:** A lower bound restricts a type parameter to be the specified type or a supertype of it, using the `super` keyword. Lower bounds are only available with wildcards.

**Technical Definition:** Unlike ordinary type variables declared in a method signature, no type inference is required when using a wildcard. Consequently, it is permissible to declare lower bounds on a wildcard, using the syntax `? super B`, where B is a lower bound. The wildcard `? super T` denotes an unknown type that is a supertype of T (or T itself). It is not possible to declare a type parameter with a lower bound in a class or method declaration.

**Beginner-Friendly Explanation:** A lower bound sets a floor — the type must be the specified type or "above" it in the hierarchy. For example, `List<? super Integer>` can accept a `List<Integer>`, `List<Number>`, or `List<Object>`. Lower bounds are useful when you want to write into a collection.

### Purposes

- To restrict type arguments to a type and its supertypes
- To enable writing (consuming) into generic structures
- To provide flexibility for consumer methods
- To support the PECS principle (Producer Extends, Consumer Super)

### Syntax Rules and Structure

```java
// Lower bounded wildcard only
List<? super T>
```

**Syntax Rules:**

- `super` can only be used with wildcards, not with type parameter declarations.
- `? super T` can write elements of type T (and subtypes).
- Can only read as `Object` from `? super T`.

**Constraints and Limitations:**

- Cannot declare a type parameter with a lower bound (e.g., `<T super Number>` is illegal).
- Cannot read as `T` from `? super T` (only as `Object`).

### Annotated Code Example

```java
// File: LowerBoundDemo.java
import java.util.*;

public class LowerBoundDemo {

    // Lower bounded wildcard: writes integers
    // Accepts List<Integer>, List<Number>, List<Object>, etc.
    public static void addIntegers(List<? super Integer> list) {
        for (int i = 1; i <= 5; i++) {
            list.add(i); // Can write Integer
        }
    }

    public static void main(String[] args) {
        List<Number> numbers = new ArrayList<>();
        addIntegers(numbers);
        System.out.println("Numbers: " + numbers); // [1, 2, 3, 4, 5]

        List<Object> objects = new ArrayList<>();
        addIntegers(objects);
        System.out.println("Objects: " + objects); // [1, 2, 3, 4, 5]

        // List<Double> doubles = new ArrayList<>();
        // addIntegers(doubles); // Compile error: Double is not a supertype of Integer
    }
}
```

**Expected Output:**
```
Numbers: [1, 2, 3, 4, 5]
Objects: [1, 2, 3, 4, 5]
```

**Step-by-Step Explanation:**

1. `addIntegers` declares `List<? super Integer>`, meaning the list must hold `Integer` or a supertype of `Integer`.
2. `list.add(i)` is legal because we can add an `Integer` to a list of `Integer`, `Number`, or `Object`.
3. `List<Number>` and `List<Object>` both work because `Number` and `Object` are supertypes of `Integer`.
4. `List<Double>` would not work because `Double` is not a supertype of `Integer`.

### Real-World Cases

- **`Collections.copy(List<? super T> dest, List<? extends T> src)`:** Classic PECS in the JDK
- **`Comparator<? super T>`:** Used in sorting methods
- **`Consumer<? super T>`:** Used in functional interfaces
- **`List<? super Integer>`:** Accepts lists of `Integer`, `Number`, or `Object`

---

## 3. Multiple Bounds

### Definitions

**Core Definition:** Multiple bounds allow a type parameter to be restricted by more than one upper bound, using the `&` separator.

**Technical Definition:** A type parameter can have multiple bounds: `<T extends B1 & B2 & B3>`. A type variable with multiple bounds is a subtype of all the types listed in the bound. If one of the bounds is a class, it must be specified first. For example, `class D<T extends A & B & C>` is valid, but `class D<T extends B & A & C>` is a compile-time error if A is a class and B, C are interfaces. The `&` separator is used instead of a comma because `<T extends C, I1, I2>` denotes three different type parameters, not one type parameter with three bounds.

**Beginner-Friendly Explanation:** Multiple bounds let you say "this type must be a subtype of all of these types." For example, `<T extends Number & Comparable<T>>` means T must be a `Number` and must be `Comparable` to itself. This lets you call both `doubleValue()` and `compareTo()` on values of type T.

### Purposes

- To require a type to satisfy multiple constraints simultaneously
- To enable calling methods from multiple bounds
- To combine class and interface bounds
- To enforce richer type constraints

### Syntax Rules and Structure

```java
<T extends ClassBound & Interface1 & Interface2>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ClassBound` | The class bound (if present, must be first) |
| `Interface1, Interface2` | Interface bounds (separated by `&`) |

**Syntax Rules:**

- At most one class may be included in the upper bounds, and if included, it must be first.
- Multiple interfaces are allowed after the class bound.
- A wildcard may not have more than one upper bound (`<? extends I1, I2>` is syntactically incorrect).
- The `&` separator is used, not a comma.

**Constraints and Limitations:**

- A class cannot be used as a second bound.
- Wildcards cannot have multiple upper bounds.

### Annotated Code Example

```java
// File: MultipleBoundsDemo.java
import java.util.*;

public class MultipleBoundsDemo {

    // Class bound first, then interfaces
    static class Animal {
        protected String name;
        public Animal(String name) { this.name = name; }
    }

    interface Swimmable {
        void swim();
    }

    interface Flyable {
        void fly();
    }

    // Multiple bounds: Animal (class) first, then Swimmable and Flyable
    static class Duck extends Animal implements Swimmable, Flyable {
        public Duck(String name) { super(name); }
        public void swim() { System.out.println(name + " swims"); }
        public void fly() { System.out.println(name + " flies"); }
    }

    // Generic method with multiple bounds
    public static <T extends Animal & Swimmable & Flyable> void describe(T animal) {
        System.out.println("Animal: " + animal.name);
        animal.swim(); // Can call Swimmable method
        animal.fly();  // Can call Flyable method
    }

    public static void main(String[] args) {
        Duck duck = new Duck("Donald");
        describe(duck);
    }
}
```

**Expected Output:**
```
Animal: Donald
Donald swims
Donald flies
```

**Step-by-Step Explanation:**

1. `Duck` extends `Animal` and implements `Swimmable` and `Flyable`.
2. The generic method declares `<T extends Animal & Swimmable & Flyable>`.
3. Inside `describe`, `animal.swim()` and `animal.fly()` are legal because T is bounded by both interfaces.
4. `animal.name` is legal because T is bounded by `Animal`.

### Real-World Cases

- **`<T extends Comparable<T> & Serializable>`:** Used when objects must be both comparable and serializable
- **`<T extends Number & Comparable<T>>`:** Used for numeric comparison algorithms
- **`<E extends Enum<E> & MyInterface>`:** Used for enums that implement additional interfaces

---

## 4. Recursive Bounds

### Definitions

**Core Definition:** A recursive bound is a type parameter bound that references the type parameter itself, enabling a class to refer to its own type in its declaration.

**Technical Definition:** A recursive type bound occurs when a type parameter's bound includes the type parameter itself. The idiom `class Node<T extends Node<T>>` is a classic example. "This idiom almost always means 'E should be the type of the subclass'". The Java `Enum` class declares `Enum<E extends Enum<E>>`, which means "an enum is an `Enum` of itself." This pattern is known as the Curiously Recurring Template Pattern (CRTP).

**Beginner-Friendly Explanation:** A recursive bound is like a self-referential instruction: "This type must be a subtype of a version of itself." For example, `Node<T extends Node<T>>` means "T must be a Node (or subclass of Node) that is parameterized with itself." This is useful for building type-safe hierarchies where a parent class needs to refer to its subclass's specific type.

### Purposes

- To allow a superclass to refer to the specific type of its subclass
- To enable type-safe method chaining and self-referential APIs
- To support the Curiously Recurring Template Pattern
- To enable the `Enum` class to declare that all enum types are `Comparable` to themselves

### Syntax Rules and Structure

```java
// Recursive bound syntax
class ClassName<T extends ClassName<T>> { ... }

// Example
class Node<T extends Node<T>> { ... }
```

**Syntax Rules:**

- The type parameter appears in its own bound.
- This enables the class to use `T` as a type that is guaranteed to be a subtype of itself.
- The pattern is often used with abstract classes and fluent APIs.

**Constraints and Limitations:**

- Recursive bounds can be difficult to read and understand; use them judiciously.
- The bound must be satisfied by any concrete subclass.

### Annotated Code Example

```java
// File: RecursiveBoundDemo.java

public class RecursiveBoundDemo {

    // Recursive bound: T must be a subclass of Node<T>
    static abstract class Node<T extends Node<T>> {
        protected String name;

        public Node(String name) {
            this.name = name;
        }

        // Returns T, which is the specific subclass type
        public abstract T self();
    }

    static class SpecialNode extends Node<SpecialNode> {
        public SpecialNode(String name) {
            super(name);
        }

        @Override
        public SpecialNode self() {
            return this; // Returns SpecialNode, not just Node
        }

        public void specialMethod() {
            System.out.println("Special method on " + name);
        }
    }

    public static void main(String[] args) {
        SpecialNode node = new SpecialNode("Test");
        // Type-safe: self() returns SpecialNode, so we can call specialMethod()
        node.self().specialMethod(); // Special method on Test
    }
}
```

**Expected Output:**
```
Special method on Test
```

**Step-by-Step Explanation:**

1. `Node<T extends Node<T>>` declares a recursive bound.
2. `SpecialNode` extends `Node<SpecialNode>`.
3. `self()` returns `T`, which is `SpecialNode` for `SpecialNode`.
4. `node.self().specialMethod()` is type-safe because `self()` returns `SpecialNode`.

### Real-World Cases

- **`Enum<E extends Enum<E>>`:** The JDK's `Enum` class uses a recursive bound
- **`Comparable<T>`:** Often used with recursive bounds in sorting
- **Builder Pattern:** Recursive bounds enable type-safe fluent builders
- **`AbstractBuilder<T extends AbstractBuilder<T>>`:** Used in Lombok's `@Builder` and similar libraries

---

## 5. Bounded Type Best Practices

### Definitions

**Core Definition:** Best practices for bounded type parameters are guidelines for choosing and using bounds effectively to maximise type safety and flexibility.

**Technical Definition:** Best practices include using bounded type parameters when the generic type must satisfy constraints, using bounded wildcards with the PECS principle for input parameters, avoiding raw types, and understanding the implications of type erasure. "Always use bounded type parameters when the generic type must satisfy constraints". "Use bounded type parameters to constrain generic types".

**Beginner-Friendly Explanation:** Best practices help you decide when to use bounds, what kind of bounds to use, and how to write code that is both safe and flexible.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Use bounded type parameters for constraints** | Restrict types when you need to call methods from the bound |
| **Use `extends` for upper bounds** | `<T extends Number>` when the type must be a Number or subclass |
| **Use PECS for wildcards** | `? extends T` for producers (reading), `? super T` for consumers (writing) |
| **Do not use bounded wildcards as return types** | If the caller has to think about wildcards, the API is probably wrong |
| **At most one class bound, first** | Multiple bounds use `&`, class must come first |
| **Use recursive bounds for type-safe self-reference** | `Node<T extends Node<T>>` for CRTP |
| **Avoid raw types** | Always use parameterized types |
| **Document type parameter bounds** | Explain why the bound exists in Javadoc |
| **Understand type erasure** | The bound becomes the erasure of the type parameter |
| **Prefer `Comparable<? super T>` over `Comparable<T>`** | More flexible for subclasses |

### Annotated Code Example

```java
// File: BoundedTypeBestPractices.java
import java.util.*;

public class BoundedTypeBestPractices {

    /**
     * Generic method with bounded type parameter.
     * @param <T> the type of elements, must be Comparable
     */
    public static <T extends Comparable<? super T>> T max(List<? extends T> list) {
        T max = list.get(0);
        for (T element : list) {
            if (element.compareTo(max) > 0) {
                max = element;
            }
        }
        return max;
    }

    // GOOD: Upper bounded wildcard for reading (producer)
    public static double sum(List<? extends Number> numbers) {
        double total = 0;
        for (Number n : numbers) {
            total += n.doubleValue();
        }
        return total;
    }

    // GOOD: Lower bounded wildcard for writing (consumer)
    public static void addNumbers(List<? super Integer> list) {
        for (int i = 1; i <= 5; i++) {
            list.add(i);
        }
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(5, 2, 8, 1, 9);
        System.out.println("Max: " + max(ints)); // 9

        System.out.println("Sum: " + sum(ints)); // 25.0

        List<Number> numbers = new ArrayList<>();
        addNumbers(numbers);
        System.out.println("Numbers: " + numbers); // [1, 2, 3, 4, 5]
    }
}
```

**Expected Output:**
```
Max: 9
Sum: 25.0
Numbers: [1, 2, 3, 4, 5]
```

**Step-by-Step Explanation:**

1. `max` uses `<T extends Comparable<? super T>>` to ensure elements can be compared.
2. `sum` uses `List<? extends Number>` for reading (PECS: producer extends).
3. `addNumbers` uses `List<? super Integer>` for writing (PECS: consumer super).

### Real-World Cases

- **`Collections.max()`:** Uses `<T extends Comparable<? super T>>`
- **`Collections.copy()`:** Uses PECS: `List<? super T> dest, List<? extends T> src`
- **`Stream.sorted()`:** Uses `Comparator<? super T>` for flexibility
- **`Optional.map()`:** Uses `Function<? super T, ? extends U>` (PECS applied to functions)

---

## References Links

- Bounded Type Parameters — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/bounded.html
- Guidelines for Wildcard Use — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/wildcardGuidelines.html
- Type Erasure — Dev.java - https://dev.java/learn/generics/type-erasure/
- Multiple Bounds — Cornell University - https://www.cs.cornell.edu/courses/JavaAndDS/files/genericUpperBdS.pdf
- Uses of Recursive Type Bounds — Stack Overflow - https://stackoverflow.com/questions/27177106/uses-of-recursive-type-bounds
- Java Generics Best Practices — Plinth (GitHub) - https://github.com/jabrena/plinth/blob/main/skills/128-java-generics/SKILL.md
- More Fun with Wildcards — IIT Bombay - https://www.cse.iitb.ac.in/~cs681/2024/resources/morefun.pdf
- Generics 2 — Virginia Tech - https://courses.cs.vt.edu/cs2114/SWDesignAndDataStructs/course-notes/MoreOnGenerics.pdf