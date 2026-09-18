# Java Bounded Wildcards: A Comprehensive Cheat Sheet

---

## Topic Overview

### Core Definition

**Java Bounded Wildcards** are a feature of Java generics that allow a type argument to be restricted to a specific type or a range of types, using the `extends` and `super` keywords with the wildcard character (`?`).

### Technical Definition

A wildcard type argument is denoted by `?`. It may be given explicit bounds, just like regular type variable declarations. An upper bound is signified by the syntax `? extends B`, where `B` is the bound. A lower bound is signified by `? super B`. The unbounded wildcard `?` is equivalent to `? extends Object`.

### Beginner-Friendly Explanation

Imagine you are organizing a bookshelf. You could say, "I only accept mystery novels" (`? extends Mystery`), meaning any subtype of mystery is fine. Or you could say, "I can put books onto any shelf that is at least a bookshelf" (`? super BookShelf`), meaning the shelf can be a generic shelf or a more specific one. Wildcards let you be flexible about types while keeping your code safe.

### Key Characteristics

- **Variance**: Upper bounded wildcards provide covariance (reading); lower bounded wildcards provide contravariance (writing).
- **Flexibility**: Wildcards allow methods to accept a wider range of parameterized types than invariant generics would allow.
- **PECS Principle**: "Producer Extends, Consumer Super" is the guiding mnemonic for choosing between `extends` and `super`.
- **Compile-time safety**: The compiler enforces wildcard constraints, preventing invalid operations.

### Prerequisites

- Basic understanding of Java generics (type parameters, generic classes, and methods).
- Familiarity with Java inheritance and interfaces.
- Knowledge of the Java Collections Framework (e.g., `List`, `Collection`).

### Related Programming Areas

- **Collections Framework**: Methods like `Collections.copy`, `Collections.sort`, and `Collections.max`.
- **Stream API**: Methods like `Stream.filter(Predicate<? super T>)` and `Stream.map(Function<? super T, ? extends R>)`.
- **API Design**: Creating flexible and type-safe APIs.
- **Design Patterns**: Strategy pattern, observer pattern, and factory methods.

### Core Concepts / Features

1. Upper Bounded Wildcards (`extends`)
2. Lower Bounded Wildcards (`super`)
3. Unbounded Wildcards
4. PECS Principle

---

## Core Concept 1: Upper Bounded Wildcards (`extends`)

### Definitions

**Core Definition**: An upper bounded wildcard restricts the unknown type to be a specific type or a subtype of that type.

**Technical Definition**: The syntax `? extends B` denotes an unknown type that is a subtype of `B` (or `B` itself, since the subtype relation is reflexive). This is used for covariance, allowing a generic type to be assigned to a broader parameterized type.

**Beginner-Friendly Explanation**: An upper bounded wildcard is like saying, "I'll accept anything that is a `Number` or a more specific kind of number, like `Integer` or `Double`." It lets you read data safely from a structure because you know the elements are at least of the bound type.

### Purposes

- To allow a method to accept a collection of a specific type or any of its subtypes.
- To enable reading data from a generic structure while preserving type safety (covariance).
- To increase API flexibility without sacrificing compile-time type checking.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
<? extends UpperBound>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `?` | The wildcard symbol. |
| `extends` | Keyword indicating an upper bound. |
| `UpperBound` | The class or interface that the unknown type must extend or implement. |

**Syntax Rules:**

- The bound can be a class or an interface.
- The wildcard `? extends Object` is equivalent to the unbounded wildcard `?`.
- You can read elements from a structure declared with `? extends T` as type `T` (or `Object`).

**Constraints and Limitations:**

- Cannot add elements to a `List<? extends T>` (except `null`) because the exact subtype is unknown.
- Cannot use `? extends T` as a return type in a method signature (this forces callers to deal with wildcards).
- The wildcard cannot have both an upper and a lower bound simultaneously.

### Annotated Complete Code Examples

**Example 1: Reading from a Collection of Subtypes**

```java
import java.util.*;

/**
 * Demonstrates upper bounded wildcards for reading data.
 */
public class UpperBoundDemo {

    /**
     * Sums all elements in a collection of Number or its subtypes.
     * @param numbers a collection of Number or any subtype
     * @return the sum as a double
     */
    public static double sumOfList(List<? extends Number> numbers) {
        double sum = 0.0;
        for (Number n : numbers) { // Reading as Number is safe
            sum += n.doubleValue();
        }
        return sum;
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3);
        List<Double> doubles = Arrays.asList(1.5, 2.5, 3.5);

        System.out.println("Sum of integers: " + sumOfList(ints));
        System.out.println("Sum of doubles: " + sumOfList(doubles));
    }
}
```

**Expected Output:**

```
Sum of integers: 6.0
Sum of doubles: 7.5
```

**Why This Output Occurs:**
- `List<? extends Number>` accepts `List<Integer>`, `List<Double>`, and any other list whose element type is a subtype of `Number`.
- Inside the method, elements are read as `Number`, so `doubleValue()` can be called safely.
- The method cannot add elements to `numbers` because the exact subtype is unknown.

**Step-by-Step Setup Guide:**
1. Create `UpperBoundDemo.java`.
2. Compile with `javac UpperBoundDemo.java`.
3. Run with `java UpperBoundDemo`.
4. Observe the output.

**Example 2: Subtyping with Upper Bounded Wildcards**

```java
import java.util.*;

/**
 * Demonstrates subtyping relationships with upper bounded wildcards.
 */
public class SubtypingDemo {

    public static void main(String[] args) {
        List<Integer> intList = new ArrayList<>();
        // List<Integer> is a subtype of List<? extends Number>
        List<? extends Number> numList = intList; // OK

        // List<? extends Integer> is a subtype of List<? extends Number>
        List<? extends Integer> intWild = intList;
        List<? extends Number> numWild = intWild; // OK

        System.out.println("Subtyping with extends works correctly.");
    }
}
```

**Expected Output:**

```
Subtyping with extends works correctly.
```

**Why This Output Occurs:**
- Although `List<Integer>` is not a subtype of `List<Number>`, it *is* a subtype of `List<? extends Number>`.
- This relationship allows code to access `Number`'s methods through `List<Integer>`'s elements, enabling covariance.

### Real-World Cases

- **`Collections.max(Collection<? extends T>)`**: Finds the maximum element in any collection of a subtype of `T`.
- **`Stream.map(Function<? super T, ? extends R>)`**: The return type of the function uses an upper bounded wildcard.
- **Reading from data sources**: Methods that process data from a list of a specific subtype (e.g., summing a list of `Integer` as `Number`).
- **API flexibility**: Accepting `List<? extends Shape>` to draw any subtype of `Shape`.

### References

- Oracle Java Tutorials – Upper Bounded Wildcards - https://docs.oracle.com/javase/tutorial/java/generics/upperBounded.html
- Java Language Specification – Wildcard Bounds - https://docs.oracle.com/javase/specs/jls/se23/html/jls-4.html#jls-4.5.1
- Oracle Java Tutorials – Wildcards and Subtyping - https://docs.oracle.com/javase/tutorial/java/generics/subtyping.html

---

## Core Concept 2: Lower Bounded Wildcards (`super`)

### Definitions

**Core Definition**: A lower bounded wildcard restricts the unknown type to be a specific type or a supertype of that type.

**Technical Definition**: The syntax `? super B` denotes an unknown type that is a supertype of `B` (or `B` itself, since the supertype relation is reflexive). This is used for contravariance, allowing a generic type to be assigned to a narrower parameterized type.

**Beginner-Friendly Explanation**: A lower bounded wildcard is like saying, "I'll accept any container that can hold a `Integer`, including a container of `Number` or `Object`." It lets you write data into a structure safely because you know the structure can accept at least the bound type.

### Purposes

- To allow a method to accept a collection that can hold a specific type or any of its supertypes.
- To enable writing data into a generic structure while preserving type safety (contravariance).
- To increase API flexibility for consumer operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
<? super LowerBound>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `?` | The wildcard symbol. |
| `super` | Keyword indicating a lower bound. |
| `LowerBound` | The class or interface that the unknown type must be a supertype of. |

**Syntax Rules:**

- The bound can be a class or an interface.
- You can add elements of type `T` to a structure declared with `? super T`.
- You cannot read elements from a `List<? super T>` as type `T` (they can only be read as `Object`).

**Constraints and Limitations:**

- Cannot read elements as the bound type from a `List<? super T>` (they are only guaranteed to be `Object`).
- Cannot use `? super T` as a return type in a method signature.
- The wildcard cannot have both an upper and a lower bound simultaneously.

### Annotated Complete Code Examples

**Example 1: Writing to a Collection of Supertypes**

```java
import java.util.*;

/**
 * Demonstrates lower bounded wildcards for writing data.
 */
public class LowerBoundDemo {

    /**
     * Adds integers 1 through n to a collection that can hold Integer or its supertypes.
     * @param list a collection of Integer or any supertype of Integer
     * @param n the number of elements to add
     */
    public static void addIntegers(List<? super Integer> list, int n) {
        for (int i = 1; i <= n; i++) {
            list.add(i); // Writing Integer is safe
        }
    }

    public static void main(String[] args) {
        List<Number> numberList = new ArrayList<>();
        List<Object> objectList = new ArrayList<>();

        addIntegers(numberList, 3);
        addIntegers(objectList, 3);

        System.out.println("Number list: " + numberList);
        System.out.println("Object list: " + objectList);
    }
}
```

**Expected Output:**

```
Number list: [1, 2, 3]
Object list: [1, 2, 3]
```

**Why This Output Occurs:**
- `List<? super Integer>` accepts `List<Number>`, `List<Object>`, and any other list whose element type is a supertype of `Integer`.
- Inside the method, `Integer` values can be added safely because the list is guaranteed to accept `Integer` or a supertype.
- The method cannot read elements as `Integer` because the exact supertype is unknown.

**Step-by-Step Setup Guide:**
1. Create `LowerBoundDemo.java`.
2. Compile with `javac LowerBoundDemo.java`.
3. Run with `java LowerBoundDemo`.
4. Observe the output.

**Example 2: Using `super` with `TreeSet` Comparator**

```java
import java.util.*;

/**
 * Demonstrates lower bounded wildcards with a Comparator.
 */
public class ComparatorDemo {

    public static void main(String[] args) {
        // A Comparator<Object> can compare any objects
        Comparator<Object> comp = (a, b) -> a.toString().compareTo(b.toString());

        // TreeSet<String> accepts a Comparator<? super String>
        TreeSet<String> set = new TreeSet<>(comp);
        set.add("banana");
        set.add("apple");
        set.add("cherry");

        System.out.println("Sorted set: " + set);
    }
}
```

**Expected Output:**

```
Sorted set: [apple, banana, cherry]
```

**Why This Output Occurs:**
- The `TreeSet` constructor accepts `Comparator<? super E>`, where `E` is the element type.
- `Comparator<Object>` is a `Comparator<? super String>` because `Object` is a supertype of `String`.
- The comparator is used to sort the strings lexicographically.

### Real-World Cases

- **`Collections.copy(List<? super T> dest, List<? extends T> src)`**: The destination list uses a lower bounded wildcard.
- **`Stream.filter(Predicate<? super T>)`**: The predicate accepts a supertype of the stream element type.
- **`TreeSet(Comparator<? super E> comparator)`**: The constructor accepts a comparator for a supertype of the element type.
- **Writing to data sinks**: Methods that push data into a structure that can accept a broader type.

### References

- Oracle Java Tutorials – Lower Bounded Wildcards - https://docs.oracle.com/javase/tutorial/java/generics/lowerBounded.html
- Java Language Specification – Wildcard Bounds - https://docs.oracle.com/javase/specs/jls/se23/html/jls-4.html#jls-4.5.1
- Oracle Java Tutorials – More Fun with Wildcards - https://docs.oracle.com/javase/tutorial/extra/generics/morefun.html

---

## Core Concept 3: Unbounded Wildcards

### Definitions

**Core Definition**: An unbounded wildcard, denoted by `?`, represents an unknown type and is technically equivalent to `? extends Object`.

**Technical Definition**: The unbounded wildcard type is specified using the wildcard character (`?`), for example, `List<?>`. This is called a list of unknown type. The wildcard `? extends Object` is equivalent to the unbounded wildcard `?`.

**Beginner-Friendly Explanation**: An unbounded wildcard is like saying, "I don't care what type it is, as long as it's an object." It's the most flexible wildcard, but it gives you the least information about the elements.

### Purposes

- To write methods that can be implemented using functionality provided in the `Object` class.
- To use methods in the generic class that do not depend on the type parameter (e.g., `List.size`, `List.clear`).
- To accept any type of collection when the specific type is irrelevant.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
<?>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `?` | The unbounded wildcard symbol. |

**Syntax Rules:**

- `List<?>` is a list of unknown type.
- `List<Object>` and `List<?>` are not the same: you can insert any `Object` into a `List<Object>`, but you can only insert `null` into a `List<?>`.
- `Class<?>` is commonly used because most methods in `Class<T>` do not depend on `T`.

**Constraints and Limitations:**

- Cannot add elements to a `List<?>` (except `null`).
- Cannot read elements as any specific type (only as `Object`).
- Cannot be used as a return type in a method signature.

### Annotated Complete Code Examples

**Example 1: Printing Any List**

```java
import java.util.*;

/**
 * Demonstrates unbounded wildcards for printing any list.
 */
public class UnboundedWildcardDemo {

    /**
     * Prints all elements of any list.
     * @param list a list of any type
     */
    public static void printList(List<?> list) {
        for (Object elem : list) { // Reading as Object is safe
            System.out.print(elem + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3);
        List<String> strings = Arrays.asList("one", "two", "three");

        printList(ints);
        printList(strings);
    }
}
```

**Expected Output:**

```
1 2 3
one two three
```

**Why This Output Occurs:**
- `List<?>` accepts a list of any type.
- Inside the method, elements are read as `Object`, which is always safe.
- The method cannot add elements to `list` (except `null`) because the exact type is unknown.

**Step-by-Step Setup Guide:**
1. Create `UnboundedWildcardDemo.java`.
2. Compile with `javac UnboundedWildcardDemo.java`.
3. Run with `java UnboundedWildcardDemo`.
4. Observe the output.

**Example 2: Unbounded Wildcard with `Class<?>`**

```java
/**
 * Demonstrates unbounded wildcards with Class<?>.
 */
public class ClassWildcardDemo {

    public static void printClassName(Class<?> clazz) {
        System.out.println("Class name: " + clazz.getName());
        System.out.println("Simple name: " + clazz.getSimpleName());
    }

    public static void main(String[] args) {
        printClassName(String.class);
        printClassName(Integer.class);
        printClassName(Double.class);
    }
}
```

**Expected Output:**

```
Class name: java.lang.String
Simple name: String
Class name: java.lang.Integer
Simple name: Integer
Class name: java.lang.Double
Simple name: Double
```

**Why This Output Occurs:**
- `Class<?>` accepts any `Class` object.
- The methods `getName()` and `getSimpleName()` do not depend on the type parameter `T`, so an unbounded wildcard is sufficient.
- This is a common pattern in reflection-based code.

### Real-World Cases

- **`Collections.swap(List<?> list, int i, int j)`**: Swaps two elements in any list.
- **`Class<?>`**: Used in reflection APIs where the specific type is unknown.
- **`Optional<?>`**: Used when the type of the optional value is unknown.
- **Logging and debugging**: Printing collections of any type.

### References

- Oracle Java Tutorials – Unbounded Wildcards - https://docs.oracle.com/javase/tutorial/java/generics/unboundedWildcards.html
- Java Language Specification – Wildcard Bounds - https://docs.oracle.com/javase/specs/jls/se23/html/jls-4.html#jls-4.5.1
- Oracle Java Tutorials – Guidelines for Wildcard Use - https://docs.oracle.com/javase/tutorial/java/generics/wildcardGuidelines.html

---

## Core Concept 4: PECS Principle

### Definitions

**Core Definition**: PECS stands for "Producer Extends, Consumer Super," a mnemonic to determine whether to use `extends` or `super` in wildcard declarations.

**Technical Definition**: The PECS principle, introduced by Joshua Bloch in *Effective Java*, states: "If a parameterized type represents a `T` producer, use `<? extends T>`; if it represents a `T` consumer, use `<? super T>`". An "in" variable (producer) serves up data to the code, while an "out" variable (consumer) holds data for use elsewhere.

**Beginner-Friendly Explanation**: PECS is a rule of thumb: if your code is *reading* data from a structure (it's a producer of data), use `extends`. If your code is *writing* data into a structure (it's a consumer of data), use `super`. If you're doing both, don't use a wildcard at all.

### Purposes

- To provide a simple mnemonic for choosing between upper and lower bounded wildcards.
- To ensure API flexibility while maintaining type safety.
- To guide developers in designing generic methods that are both flexible and safe.

### Syntax Rules and Structure

The PECS principle is a design guideline, not a syntax rule. However, it maps directly to the wildcard syntax:

| Role | Wildcard | Keyword |
|------|----------|---------|
| Producer (reads data) | `<? extends T>` | `extends` |
| Consumer (writes data) | `<? super T>` | `super` |
| Both producer and consumer | No wildcard (use `<T>`) | N/A |

**Syntax Rules:**

- An "in" variable is defined with an upper bounded wildcard, using the `extends` keyword.
- An "out" variable is defined with a lower bounded wildcard, using the `super` keyword.
- In the case where the "in" variable can be accessed using methods defined in the `Object` class, use an unbounded wildcard.
- In the case where the code needs to access the variable as both an "in" and an "out" variable, do not use a wildcard.

**Constraints and Limitations:**

- PECS does not apply to a method's return type. Using a wildcard as a return type should be avoided because it forces programmers using the code to deal with wildcards.
- PECS is a guideline, not a strict rule; there are cases where no wildcard is appropriate.

### Annotated Complete Code Examples

**Example 1: PECS in a Copy Method**

```java
import java.util.*;

/**
 * Demonstrates the PECS principle with a copy method.
 */
public class PecsDemo {

    /**
     * Copies elements from src to dest.
     * src is a producer (in) -> extends
     * dest is a consumer (out) -> super
     */
    public static <T> void copy(List<? extends T> src, List<? super T> dest) {
        for (T element : src) { // Reading from src (producer)
            dest.add(element);   // Writing to dest (consumer)
        }
    }

    public static void main(String[] args) {
        List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
        List<Number> numbers = new ArrayList<>();

        copy(integers, numbers); // Integer extends Number
        System.out.println("Numbers: " + numbers);

        List<Object> objects = new ArrayList<>();
        copy(integers, objects); // Integer extends Object
        System.out.println("Objects: " + objects);
    }
}
```

**Expected Output:**

```
Numbers: [1, 2, 3, 4, 5]
Objects: [1, 2, 3, 4, 5]
```

**Why This Output Occurs:**
- `src` is `List<? extends T>`, so it can produce elements of type `T`. `List<Integer>` is a `List<? extends Number>`.
- `dest` is `List<? super T>`, so it can consume elements of type `T`. `List<Number>` and `List<Object>` are `List<? super Integer>`.
- The PECS principle ensures both reading from `src` and writing to `dest` are type-safe.

**Step-by-Step Setup Guide:**
1. Create `PecsDemo.java`.
2. Compile with `javac PecsDemo.java`.
3. Run with `java PecsDemo`.
4. Observe the output.

**Example 2: PECS with `Collections.max`**

```java
import java.util.*;

/**
 * Demonstrates PECS with Collections.max.
 */
public class MaxDemo {

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(3, 1, 4, 1, 5, 9);
        // Collections.max accepts Collection<? extends T>
        Integer maxInt = Collections.max(ints);
        System.out.println("Max integer: " + maxInt);

        List<String> strings = Arrays.asList("apple", "orange", "banana");
        String maxStr = Collections.max(strings);
        System.out.println("Max string: " + maxStr);
    }
}
```

**Expected Output:**

```
Max integer: 9
Max string: orange
```

**Why This Output Occurs:**
- `Collections.max` is declared as `public static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll)`.
- The parameter uses `? extends T` because the collection is a producer of elements to be compared.
- The method reads elements from the collection and returns the maximum.

### Real-World Cases

- **`Collections.copy`**: Uses PECS for its source and destination parameters.
- **`Stream.filter`**: `filter(Predicate<? super T> predicate)` uses `super` because the predicate consumes elements.
- **`Stream.map`**: `map(Function<? super T, ? extends R> mapper)` uses both `super` (for input) and `extends` (for output).
- **`TreeSet` constructor**: `TreeSet(Comparator<? super E> comparator)` uses `super` because the comparator consumes elements.
- **API design**: Following PECS leads to more flexible and reusable generic APIs.

### References

- Oracle Java Tutorials – Guidelines for Wildcard Use - https://docs.oracle.com/javase/tutorial/java/generics/wildcardGuidelines.html
- Effective Java (Third Edition) by Joshua Bloch – Item 31: Use bounded wildcards to increase API flexibility
- Java Language Specification – Wildcards - https://docs.oracle.com/javase/specs/jls/se23/html/jls-4.html#jls-4.5.1
- Oracle Java Tutorials – More Fun with Wildcards - https://docs.oracle.com/javase/tutorial/extra/generics/morefun.html

---

## References

- Oracle Java Tutorials – Wildcards - https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html
- Oracle Java Tutorials – Upper Bounded Wildcards - https://docs.oracle.com/javase/tutorial/java/generics/upperBounded.html
- Oracle Java Tutorials – Lower Bounded Wildcards - https://docs.oracle.com/javase/tutorial/java/generics/lowerBounded.html
- Oracle Java Tutorials – Unbounded Wildcards - https://docs.oracle.com/javase/tutorial/java/generics/unboundedWildcards.html
- Oracle Java Tutorials – Guidelines for Wildcard Use - https://docs.oracle.com/javase/tutorial/java/generics/wildcardGuidelines.html
- Oracle Java Tutorials – Wildcards and Subtyping - https://docs.oracle.com/javase/tutorial/java/generics/subtyping.html
- Oracle Java Tutorials – More Fun with Wildcards - https://docs.oracle.com/javase/tutorial/extra/generics/morefun.html
- Java Language Specification – Wildcards (JLS §4.5.1) - https://docs.oracle.com/javase/specs/jls/se23/html/jls-4.html#jls-4.5.1
- Java API Documentation – java.util.Collections - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Collections.html
- Effective Java (Third Edition) by Joshua Bloch – Item 31: Use bounded wildcards to increase API flexibility