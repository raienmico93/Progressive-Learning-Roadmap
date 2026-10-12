# Java Wildcards: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A wildcard in Java generics is the question mark (`?`) that represents an unknown type, used to increase the flexibility of generic APIs by allowing methods to accept a wider range of parameterized types.

**Technical Definition:** Per the Java Language Specification, a wildcard is a restricted form of existential type denoted by `?` with optional bounds. Wildcards may be given explicit bounds, just like regular type variable declarations. An upper bound is signified by `? extends B`, and a lower bound by `? super B`. The wildcard `? extends Object` is equivalent to the unbounded wildcard `?`. Critically, "the wildcard is never used as a type argument for a generic method invocation, a generic class instance creation, or a supertype".

**Beginner-Friendly Explanation:** A wildcard is like saying "any type goes" or "some type in this family." Without wildcards, `List<String>` and `List<Integer>` are completely unrelated types — even though `String` and `Integer` are both `Object`s. Wildcards solve this by letting you write code that works with "a list of some unknown type" (`List<?>`), "a list of numbers or their subclasses" (`List<? extends Number>`), or "a list of integers or their superclasses" (`List<? super Integer>`). Think of a wildcard as a flexible placeholder that says "I don't know the exact type, but I know its boundaries."

### Key Characteristics

- **Existential Type:** A wildcard represents an unknown type, not a specific type.
- **Three Forms:** Unbounded (`?`), upper bounded (`? extends T`), and lower bounded (`? super T`).
- **Read/Write Asymmetry:** Upper bounded wildcards allow reading (producing), lower bounded wildcards allow writing (consuming).
- **Never a Type Argument in Creation:** Wildcards cannot be used in `new ArrayList<?>()` or as type witnesses in method calls.
- **Capture Conversion:** The compiler can "capture" a wildcard as a fresh type variable in certain contexts.
- **Invariance Bridge:** Wildcards create subtyping relationships between parameterized types that would otherwise be invariant.

### Prerequisites

- Basic understanding of Java generics (type parameters, generic classes, generic methods)
- Familiarity with the Java Collections Framework (`List`, `Set`, `Map`)
- Knowledge of Java inheritance and subtyping
- Awareness of the PECS principle (Producer Extends, Consumer Super)
- Java 5 or later

### Related Programming Areas

- **Generic Methods:** Wildcards and generic methods work together to create flexible APIs.
- **Collections Framework:** Methods like `Collections.copy()`, `Collections.max()`, and `List.addAll()` use wildcards extensively.
- **Functional Interfaces:** `Comparator<? super T>`, `Function<? super T, ? extends R>`, and `Consumer<? super T>` all use wildcards.
- **API Design:** Bounded wildcards increase API flexibility by allowing callers to pass a wider range of types.
- **Stream API:** Many stream operations use wildcards in their method signatures.

---

## Core Concepts / Features

---

## 1. Unbounded Wildcards

### Definitions

**Core Definition:** The unbounded wildcard `?` represents an unknown type and is used when the type is not needed or when only methods from `Object` are used.

**Technical Definition:** The unbounded wildcard type is specified using the wildcard character (`?`), for example, `List<?>`. This is called a "list of unknown type." The wildcard `? extends Object` is equivalent to the unbounded wildcard `?`. There are two scenarios where an unbounded wildcard is useful: if you are writing a method that can be implemented using functionality provided in the `Object` class, or when the code is using methods in the generic class that do not depend on the type parameter.

**Beginner-Friendly Explanation:** An unbounded wildcard says "I don't care what type this is." For example, `List<?>` means "a list of anything." You can read elements from it (as `Object`), but you cannot add elements to it (except `null`). It is useful for methods like `printList` that just need to iterate over a collection and call `toString()`.

### Purposes

- To write methods that work with any parameterized type
- To use methods that do not depend on the type parameter (e.g., `size()`, `clear()`)
- To provide maximum flexibility when the type is irrelevant to the operation
- To avoid the restrictions of `List<Object>` (which only accepts `Object` and its subtypes)

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Unbounded wildcard
List<?>
Collection<?>
Map<?, ?>
```

**Component Breakdown:**

| Form | Description |
|------|-------------|
| `List<?>` | A list of unknown element type |
| `Collection<?>` | A collection of unknown element type |
| `Map<?, ?>` | A map with unknown key and value types |

**Syntax Rules:**

- `?` stands alone without `extends` or `super`.
- `List<?>` is the supertype of all `List<X>` types for any concrete `X`.
- You can read elements as `Object`.
- You can add only `null`.
- You can invoke methods that do not depend on the type parameter (`size()`, `clear()`, `iterator()`).

**Constraints and Limitations:**

- Cannot add elements (except `null`) because the exact element type is unknown.
- Cannot read as a specific type (only as `Object`).
- Cannot be used in `new ArrayList<?>()` (illegal).

### Annotated Complete Step-by-Step Code Examples

**Example 1: Unbounded Wildcard for Printing**

```java
// File: UnboundedWildcardDemo.java
import java.util.*;

public class UnboundedWildcardDemo {

    // Unbounded wildcard: accepts a list of any type
    public static void printList(List<?> list) {
        for (Object element : list) {
            System.out.print(element + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3);
        List<String> strings = Arrays.asList("A", "B", "C");
        List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);

        printList(ints);     // 1 2 3
        printList(strings);  // A B C
        printList(doubles);  // 1.1 2.2 3.3
    }
}
```

**Expected Output:**
```
1 2 3 
A B C 
1.1 2.2 3.3 
```

**Step-by-Step Explanation:**

1. `printList` declares `List<?>`, meaning "a list of unknown type."
2. The method iterates over elements as `Object` and prints them.
3. `List<Integer>`, `List<String>`, and `List<Double>` are all subtypes of `List<?>`.
4. The method works for any list type without modification.

**Example 2: Reading vs. Writing with Unbounded Wildcard**

```java
// File: UnboundedReadWriteDemo.java
import java.util.*;

public class UnboundedReadWriteDemo {
    public static void main(String[] args) {
        List<?> list = new ArrayList<String>();

        // Reading is safe (as Object)
        Object first = list.get(0);  // This would throw IndexOutOfBoundsException if empty
        System.out.println("Read: " + first);

        // Writing is NOT safe (except null)
        // list.add("Hello");  // Compile error: cannot add to List<?>
        list.add(null);        // Only null is allowed
    }
}
```

**Expected Output:**
```
Read: null
```

**Step-by-Step Explanation:**

1. `List<?> list` accepts a `List<String>`.
2. `list.get(0)` returns an `Object` (which is `null` because the list is empty).
3. `list.add("Hello")` is a compile-time error because the exact type is unknown.
4. `list.add(null)` is allowed because `null` is a member of every type.

### Real-World Cases

- **`printList()` methods:** Printing any collection regardless of element type
- **`Class<?>`:** The `Class` class is almost always used with an unbounded wildcard because most of its methods do not depend on `T`
- **`Collection<?>` parameters:** Accepting any collection in utility methods
- **`Map<?, ?>`:** Working with maps where key and value types are irrelevant

---

## 2. Upper Bounded Wildcards

### Definitions

**Core Definition:** An upper bounded wildcard `? extends T` restricts the unknown type to be `T` or a subtype of `T`.

**Technical Definition:** To declare an upper-bounded wildcard, use the wildcard character (`?`), followed by the `extends` keyword, followed by its upper bound. Note that, in this context, `extends` is used in a general sense to mean either "extends" (as in classes) or "implements" (as in interfaces). The upper bounded wildcard `<? extends Foo>`, where `Foo` is any type, matches `Foo` and any subtype of `Foo`.

**Beginner-Friendly Explanation:** An upper bounded wildcard says "this type must be T or a subclass of T." For example, `List<? extends Number>` means "a list of `Number` or its subclasses (`Integer`, `Double`, etc.)." This is useful when you want to **read** values from a collection because you know they are at least of type `T`.

### Purposes

- To relax the restrictions on a variable to accept a family of related types
- To enable reading (producing) from generic structures
- To create subtyping relationships between parameterized types
- To increase API flexibility for producer methods

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Upper bounded wildcard
List<? extends Number>
Collection<? extends Shape>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `?` | The wildcard |
| `extends` | The keyword indicating an upper bound |
| `Number` | The upper bound type |

**Syntax Rules:**

- `List<? extends Number>` matches `List<Number>`, `List<Integer>`, `List<Double>`, etc.
- You can read elements as type `T` (`Number` in this example).
- You cannot add elements (except `null`) because the exact subtype is unknown.
- `List<? extends Integer>` is a subtype of `List<? extends Number>` because `Integer` extends `Number`.

**Constraints and Limitations:**

- Cannot add elements (except `null`).
- Cannot be used as a return type without forcing callers to deal with wildcards.
- Overusing upper bounded wildcards can make APIs harder to understand.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Upper Bounded Wildcard for Summing**

```java
// File: UpperBoundedWildcardDemo.java
import java.util.*;

public class UpperBoundedWildcardDemo {

    // Upper bounded wildcard: accepts List<Number> or any subclass
    public static double sumOfList(List<? extends Number> list) {
        double sum = 0;
        for (Number n : list) {
            sum += n.doubleValue();  // Can read as Number
        }
        return sum;
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3);
        List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);

        System.out.println("Sum ints: " + sumOfList(ints));     // 6.0
        System.out.println("Sum doubles: " + sumOfList(doubles)); // 6.6

        // Without the wildcard, this would NOT compile:
        // public static double sumOfList(List<Number> list) { ... }
        // sumOfList(ints); // Compile error: List<Integer> is not a List<Number>
    }
}
```

**Expected Output:**
```
Sum ints: 6.0
Sum doubles: 6.6
```

**Step-by-Step Explanation:**

1. `sumOfList` declares `List<? extends Number>`.
2. `List<Integer>` and `List<Double>` are both subtypes of `List<? extends Number>`.
3. Inside the method, `n` is treated as `Number`, so `doubleValue()` is available.
4. Without the wildcard, `sumOfList(List<Number>)` would reject `List<Integer>`.

### Real-World Cases

- **`Collections.max(List<? extends T>)`:** Used to find the maximum element
- **`Stream.map(Function<? super T, ? extends R>)`:** The return type is an upper bounded wildcard
- **`Optional.map(Function<? super T, ? extends U>)`:** Similar PECS pattern
- **Reading from collections:** Any method that only reads from a collection should use `? extends T`

---

## 3. Lower Bounded Wildcards

### Definitions

**Core Definition:** A lower bounded wildcard `? super T` restricts the unknown type to be `T` or a supertype of `T`.

**Technical Definition:** A lower bounded wildcard is expressed using the wildcard character (`?`), followed by the `super` keyword, followed by its lower bound: `<? super A>`. You can specify an upper bound for a wildcard, or you can specify a lower bound, but you cannot specify both. The syntax `? super T` denotes an unknown type that is a supertype of `T` (or `T` itself).

**Beginner-Friendly Explanation:** A lower bounded wildcard says "this type must be T or a superclass of T." For example, `List<? super Integer>` means "a list of `Integer` or its superclasses (`Number`, `Object`)." This is useful when you want to **write** values into a collection because you know the collection can hold at least `T` values.

### Purposes

- To enable writing (consuming) into generic structures
- To maximize flexibility for consumer methods
- To allow a method to accept a collection of a supertype of the elements it writes
- To support the PECS principle (Consumer Super)

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Lower bounded wildcard
List<? super Integer>
Collection<? super String>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `?` | The wildcard |
| `super` | The keyword indicating a lower bound |
| `Integer` | The lower bound type |

**Syntax Rules:**

- `List<? super Integer>` matches `List<Integer>`, `List<Number>`, and `List<Object>`.
- You can add elements of type `T` (and subtypes).
- You can only read elements as `Object`.
- Cannot be used as a type parameter in a class or method declaration (only in wildcards).

**Constraints and Limitations:**

- Cannot read as `T` (only as `Object`).
- Cannot specify both `extends` and `super` on the same wildcard.
- Using lower bounded wildcards as return types is discouraged.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Lower Bounded Wildcard for Adding**

```java
// File: LowerBoundedWildcardDemo.java
import java.util.*;

public class LowerBoundedWildcardDemo {

    // Lower bounded wildcard: accepts List<Integer>, List<Number>, List<Object>
    public static void addIntegers(List<? super Integer> list) {
        for (int i = 1; i <= 5; i++) {
            list.add(i);  // Can write Integer
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

1. `addIntegers` declares `List<? super Integer>`.
2. `List<Number>` and `List<Object>` are both subtypes of `List<? super Integer>` because `Number` and `Object` are supertypes of `Integer`.
3. Inside the method, `list.add(i)` is legal because we can add an `Integer` to a list of `Integer`, `Number`, or `Object`.
4. `List<Double>` would not work because `Double` is not a supertype of `Integer`.

### Real-World Cases

- **`Collections.copy(List<? super T> dest, List<? extends T> src)`:** Classic PECS example
- **`TreeSet(Comparator<? super E> c)`:** Accepts comparators of supertypes
- **`Stream.collect(Collector<? super T, A, R>)`:** Uses `? super T`
- **Writing to collections:** Any method that only writes to a collection should use `? super T`

---

## 4. Wildcard Capture

### Definitions

**Core Definition:** Wildcard capture is the compiler's process of inferring a specific type for a wildcard in a particular expression, temporarily treating the unknown type as a fresh type variable.

**Technical Definition:** In some cases, the compiler infers the type of a wildcard. For example, a list may be defined as `List<?>` but, when evaluating an expression, the compiler infers a particular type from the code. This scenario is known as wildcard capture. For the most part, you do not need to worry about wildcard capture, except when you see an error message that contains the phrase "capture of".

**Beginner-Friendly Explanation:** Wildcard capture is when the compiler says "I don't know what type this list holds, but for the purpose of this operation, I'll treat it as a specific (but unknown) type." This usually works fine, but sometimes the compiler gets confused and produces a "capture of" error. The solution is to use a helper method to "capture" the wildcard as a named type variable.

### Purposes

- To allow the compiler to perform type-safe operations on wildcard types
- To work around capture errors using helper methods
- To enable operations like `swap` that need to temporarily store an element of unknown type

### Syntax Rules and Structure

**The Problem (Capture Error):**

```java
public class WildcardError {
    void foo(List<?> i) {
        i.set(0, i.get(0));  // Capture error!
    }
}
```

**The Solution (Helper Method):**

```java
public class WildcardFixed {
    void foo(List<?> i) {
        fooHelper(i);
    }

    private <T> void fooHelper(List<T> l) {
        l.set(0, l.get(0));  // Works!
    }
}
```

**Syntax Rules:**

- The helper method declares a type parameter `<T>` and takes `List<T>`.
- The wildcard `List<?>` is passed to the helper, and the compiler captures the wildcard as `T`.
- By convention, helper methods are named `originalMethodNameHelper`.

**Constraints and Limitations:**

- Capture errors occur when the compiler cannot prove that an operation is type-safe.
- Helper methods are the standard workaround.

### Annotated Code Example

```java
// File: WildcardCaptureDemo.java
import java.util.*;

public class WildcardCaptureDemo {

    // This method produces a capture error:
    // public static void swapFirst(List<?> list) {
    //     list.set(0, list.get(1));  // Capture error!
    // }

    // Fixed with a helper method
    public static void swapFirst(List<?> list) {
        swapFirstHelper(list);
    }

    private static <T> void swapFirstHelper(List<T> list) {
        T temp = list.get(0);
        list.set(0, list.get(1));
        list.set(1, temp);
    }

    public static void main(String[] args) {
        List<Integer> ints = new ArrayList<>(Arrays.asList(1, 2, 3));
        swapFirst(ints);
        System.out.println("Swapped: " + ints); // [2, 1, 3]

        List<String> strings = new ArrayList<>(Arrays.asList("A", "B", "C"));
        swapFirst(strings);
        System.out.println("Swapped: " + strings); // [B, A, C]
    }
}
```

**Expected Output:**
```
Swapped: [2, 1, 3]
Swapped: [B, A, C]
```

**Step-by-Step Explanation:**

1. `swapFirst(List<?>)` cannot directly perform `list.set(0, list.get(1))` because the compiler does not know the exact type.
2. The helper method `swapFirstHelper(List<T>)` captures the wildcard as `T`.
3. Inside the helper, `T temp = list.get(0)` is type-safe because `T` is the same type as the list's element type.
4. `list.set(0, list.get(1))` is legal because `list.get(1)` returns `T` and `list.set(0, T)` accepts `T`.
5. The helper method solves the capture error.

### Real-World Cases

- **`Collections.swap()`:** The JDK implements `swap` using a similar helper pattern
- **Custom swap methods:** Any method that needs to exchange elements in a wildcard list
- **Copy methods:** Methods that copy elements between wildcard collections

---

## 5. PECS Principle (Producer Extends, Consumer Super)

### Definitions

**Core Definition:** PECS (Producer Extends, Consumer Super) is a mnemonic for choosing the correct wildcard: use `? extends T` when the collection produces (provides) values, and use `? super T` when the collection consumes (accepts) values.

**Technical Definition:** The PECS rule states that for a `T` producer, you should use `Foo<? extends T>`; for a `T` consumer, you should use `Foo<? super T>`. This rule applies only to input parameters and must be applied from the collection's point of view. If a collection is both a producer and a consumer, you should not use a wildcard at all — use the base type `T`.

**Beginner-Friendly Explanation:** If a collection is **giving you things** (you're reading from it), use `extends`. If a collection is **taking things from you** (you're writing to it), use `super`. If you're doing both, don't use a wildcard.

### Purposes

- To provide a simple, memorable rule for choosing wildcards
- To maximise API flexibility while maintaining type safety
- To avoid unnecessary restrictions in generic method signatures
- To follow established Java best practices

### Syntax Rules and Structure

| Scenario | Wildcard | Example |
|----------|----------|---------|
| Producer (reading) | `? extends T` | `List<? extends Number>` |
| Consumer (writing) | `? super T` | `List<? super Integer>` |
| Both | `T` (no wildcard) | `List<T>` |

**Syntax Rules:**

- Apply PECS from the collection's perspective, not the method's perspective.
- If a type parameter appears only once in a method declaration, replace it with a wildcard.
- Do not use wildcard types as return types.

**Constraints and Limitations:**

- Using the wrong wildcard reduces flexibility.
- Wildcards cannot be used when the type parameter appears in multiple roles (e.g., both parameter and return type).

### Annotated Code Example

```java
// File: PecsDemo.java
import java.util.*;

public class PecsDemo {

    // PRODUCER: reads from the list (uses extends)
    public static double sumOfNumbers(List<? extends Number> numbers) {
        double sum = 0;
        for (Number n : numbers) {
            sum += n.doubleValue();
        }
        return sum;
    }

    // CONSUMER: writes to the list (uses super)
    public static void addIntegers(List<? super Integer> list) {
        for (int i = 1; i <= 5; i++) {
            list.add(i);
        }
    }

    public static void main(String[] args) {
        // Producer: accepts List<Integer>, List<Double>, etc.
        List<Integer> ints = Arrays.asList(1, 2, 3);
        List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);
        System.out.println("Sum ints: " + sumOfNumbers(ints));     // 6.0
        System.out.println("Sum doubles: " + sumOfNumbers(doubles)); // 6.6

        // Consumer: accepts List<Integer>, List<Number>, List<Object>
        List<Number> numbers = new ArrayList<>();
        addIntegers(numbers);
        System.out.println("Numbers: " + numbers); // [1, 2, 3, 4, 5]
    }
}
```

**Expected Output:**
```
Sum ints: 6.0
Sum doubles: 6.6
Numbers: [1, 2, 3, 4, 5]
```

**Step-by-Step Explanation:**

1. `sumOfNumbers` uses `List<? extends Number>` because it reads from the list (producer).
2. `addIntegers` uses `List<? super Integer>` because it writes to the list (consumer).
3. The PECS rule dictates these choices: producer extends, consumer super.

### Real-World Cases

- **`Collections.copy(List<? super T> dest, List<? extends T> src)`:** The JDK's classic PECS example
- **`Stream.map(Function<? super T, ? extends R>)`:** PECS applied to functions
- **`Comparator<? super T>`:** Used in sorting methods to allow comparators of supertypes
- **`TreeSet(Comparator<? super E>)`:** PECS for constructors

---

## 6. Wildcard Best Practices

### Definitions

**Core Definition:** Best practices for wildcards are guidelines for choosing and using wildcards effectively to maximise API flexibility while maintaining type safety.

**Technical Definition:** Best practices derive from *Effective Java* and Oracle's Java Tutorials. The key principles are: use bounded wildcards to increase API flexibility; use PECS for input parameters; do not use wildcards as return types; and use unbounded wildcards when the type is irrelevant.

**Beginner-Friendly Explanation:** Best practices help you decide when to use wildcards and which type of wildcard to use, so that your APIs are both flexible and safe.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Use PECS for input parameters** | `? extends T` for producers, `? super T` for consumers |
| **Do not use wildcards as return types** | Forces callers to deal with wildcards; use a type parameter instead |
| **Use unbounded wildcards when the type is irrelevant** | `List<?>` when only `Object` methods are needed |
| **If a type parameter appears only once, use a wildcard** | "If a type parameter appears only once in a method declaration, replace it with a wildcard" |
| **Do not use wildcards when the type is both producer and consumer** | Use the base type `T` without wildcards |
| **Use helper methods for wildcard capture** | When a capture error occurs, a private helper method with a type parameter solves it |
| **Prefer `Comparator<? super T>` over `Comparator<T>`** | More flexible for callers |
| **Prefer `Function<? super T, ? extends R>` over `Function<T, R>`** | PECS applied to functional interfaces |
| **Document wildcard usage** | Explain why a wildcard is used in Javadoc |
| **Avoid overusing wildcards** | They can make APIs harder to understand |

### Annotated Code Example

```java
// File: WildcardBestPractices.java
import java.util.*;

public class WildcardBestPractices {

    // GOOD: Producer extends (reading)
    public static double sum(List<? extends Number> numbers) {
        double total = 0;
        for (Number n : numbers) total += n.doubleValue();
        return total;
    }

    // GOOD: Consumer super (writing)
    public static void addNumbers(List<? super Integer> list) {
        for (int i = 1; i <= 5; i++) list.add(i);
    }

    // GOOD: Unbounded wildcard when type is irrelevant
    public static int countElements(Collection<?> collection) {
        return collection.size();
    }

    // GOOD: PECS applied to Comparator
    public static <T> T max(List<? extends T> list, Comparator<? super T> comparator) {
        T max = list.get(0);
        for (T element : list) {
            if (comparator.compare(element, max) > 0) {
                max = element;
            }
        }
        return max;
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(5, 2, 8, 1, 9);
        System.out.println("Sum: " + sum(ints)); // 25.0

        List<Number> numbers = new ArrayList<>();
        addNumbers(numbers);
        System.out.println("Numbers: " + numbers); // [1, 2, 3, 4, 5]

        System.out.println("Count: " + countElements(ints)); // 5

        System.out.println("Max: " + max(ints, Comparator.naturalOrder())); // 9
    }
}
```

**Expected Output:**
```
Sum: 25.0
Numbers: [1, 2, 3, 4, 5]
Count: 5
Max: 9
```

**Step-by-Step Explanation:**

1. `sum` uses `List<? extends Number>` for reading (PECS: producer extends).
2. `addNumbers` uses `List<? super Integer>` for writing (PECS: consumer super).
3. `countElements` uses `Collection<?>` because it only needs `size()`.
4. `max` uses `Comparator<? super T>` because the comparator consumes `T` values.

### Real-World Cases

- **Java Collections Framework:** Uses wildcards extensively in method signatures
- **Stream API:** `Stream.map(Function<? super T, ? extends R>)` follows PECS
- **Spring Framework:** Generic APIs use wildcards for flexibility
- **Guava:** `ImmutableList<? extends E>` and similar patterns

---

## Comparison Table: Wildcard Types

| Feature | Unbounded `?` | Upper Bounded `? extends T` | Lower Bounded `? super T` |
|---------|--------------|---------------------------|--------------------------|
| **Meaning** | Any type | T or subtype of T | T or supertype of T |
| **Reading** | As `Object` | As `T` | As `Object` |
| **Writing** | Only `null` | Only `null` | Can write `T` and subtypes |
| **Use Case** | Type irrelevant | Producer (reading) | Consumer (writing) |
| **Example** | `List<?>` | `List<? extends Number>` | `List<? super Integer>` |
| **Subtyping** | Supertype of all `List<X>` | Covariant | Contravariant |
| **PECS** | N/A | Producer Extends | Consumer Super |

---

## References Links

- Guidelines for Wildcard Use — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/wildcardGuidelines.html
- Wildcards — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html
- Unbounded Wildcards — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/unboundedWildcards.html
- Upper Bounded Wildcards — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/upperBounded.html
- Lower Bounded Wildcards — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/lowerBounded.html
- Wildcard Capture and Helper Methods — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/capture.html
- Wildcards and Subtyping — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/subtyping.html
- Java Generics PECS – Producer Extends Consumer Super — Baeldung - https://www.baeldung.com/java-generics-pecs
- Java Language Specification, §4.5.1: Type Arguments and Wildcards — Oracle - https://docs.oracle.com/javase/specs/jls/se23/jls23.pdf
- Effective Java (3rd Edition) by Joshua Bloch — Item 31: Use bounded wildcards to increase API flexibility
- Java Generics Best Practices — Plinth (GitHub) - https://github.com/jabrena/plinth/blob/main/skills/128-java-generics/SKILL.md
- Java Generics Wildcards Explained: extends, super, PECS, and Best Practices — Cleverence - https://www.cleverence.com/articles/oracle-documentation/java-generics-wildcards-4729/