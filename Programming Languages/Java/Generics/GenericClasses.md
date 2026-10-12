# Java Generic Classes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A generic class is a class that is parameterized over one or more types, allowing it to operate on different types while providing compile-time type safety.

**Technical Definition:** Per the Java Language Specification, a generic class is a class declaration that declares one or more type variables, called type parameters. The type parameters are enclosed in angle brackets (`<>`) following the class name. A generic type may be declared with multiple type parameters, and type parameter declarations can constrain the set of types allowed by using the `extends` keyword. Unlike template mechanisms, there is only one type and one class file corresponding to the generic class, regardless of how many different instantiations a program has.

**Beginner-Friendly Explanation:** A generic class is like a blueprint for a container that can hold any type of item. Instead of building separate containers for strings, integers, and employees, you build one container with a placeholder — `T` — that gets filled in when you actually use the class. For example, `Box<String>` is a box that holds strings, and `Box<Integer>` is a box that holds integers, both from the same generic `Box<T>` class. This way, the compiler ensures you only put strings in a `Box<String>` and only get strings out.

### Key Characteristics

- **Type Parameterization:** Classes are parameterized over one or more types, with type parameters declared in angle brackets.
- **Single Class File:** Only one class file exists for a generic class, regardless of how many different type arguments are used.
- **Compile-Time Type Safety:** The compiler enforces type constraints, preventing runtime `ClassCastException`.
- **Type Erasure:** Generic type information is removed at runtime; type parameters are replaced with their bounds or `Object`.
- **Invariance:** `Box<Integer>` is not a subtype of `Box<Number>`, even though `Integer` is a subtype of `Number`.
- **No Static Type Parameters:** Type parameters cannot be referenced in static methods, initializers, or static variable declarations.

### Prerequisites

- Basic understanding of Java classes, interfaces, and methods
- Familiarity with the Java Collections Framework (`List`, `Set`, `Map`)
- Knowledge of inheritance and subtyping
- Awareness of the `Object` class as the root of the hierarchy
- Java 5 or later

### Related Programming Areas

- **Collections Framework:** Generics are pervasive throughout `List<E>`, `Set<E>`, `Map<K,V>`
- **Stream API:** Streams rely heavily on generics for type-safe pipelines
- **Design Patterns:** Generic types enable type-safe builders, factories, and repositories
- **Functional Programming:** Generic functional interfaces (`Function<T,R>`, `Predicate<T>`)
- **API Design:** Generic classes form the backbone of reusable library components

---

## Core Concepts / Features

---

## 1. Generic Class Declaration

### Definitions

**Core Definition:** A generic class declaration is the syntactic construct that introduces a class with one or more type parameters.

**Technical Definition:** A generic class is defined with the following format: `class name<T1, T2, ..., Tn> { /* ... */ }`. The type parameter section, delimited by angle brackets (`<>`), follows the class name. It specifies the type parameters (also called type variables) T1, T2, ..., and Tn.

**Beginner-Friendly Explanation:** A generic class declaration is like a form with blanks. You write `class Box<T>`, where `T` is the blank that will be filled in with a specific type when you use the class.

### Purposes

- To create reusable container classes that work with any type
- To provide compile-time type safety for collections and data structures
- To eliminate the need for casting when retrieving elements
- To enable generic algorithms that adapt to the types they receive

### Syntax Rules and Structure

**Complete General Syntax:**

```java
[accessModifier] class ClassName<T1, T2, ..., Tn> {
    // fields, methods, constructors using type parameters
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `class` | Keyword introducing a class declaration |
| `ClassName` | The name of the class |
| `<T1, T2, ..., Tn>` | Type parameter section (angle-bracketed, comma-separated) |
| `{ ... }` | Class body using type parameters as types |

**Syntax Rules:**

- Type parameters are declared after the class name in angle brackets.
- Type parameters can be used anywhere inside the class where a type is expected.
- A class can have one or more type parameters.
- Type parameters cannot be primitive types (use wrapper types instead).
- Type parameters cannot be used in static contexts.

**Constraints and Limitations:**

- Cannot instantiate type parameters directly (`new T()` is illegal).
- Cannot create arrays of parameterized types (`new List<String>[10]` is illegal).
- Cannot use `instanceof` with parameterized types.

### Annotated Complete Step-by-Step Code Examples

**Example 1: A Simple Generic Box Class**

```java
// File: GenericBoxDemo.java
public class GenericBoxDemo {

    // Generic class with type parameter T
    static class Box<T> {
        private T content;  // T is used as a field type

        public void set(T content) {  // T is used as a parameter type
            this.content = content;
        }

        public T get() {  // T is used as a return type
            return content;
        }
    }

    public static void main(String[] args) {
        // Box for String
        Box<String> stringBox = new Box<>();
        stringBox.set("Hello Generics");
        String str = stringBox.get();  // No cast needed
        System.out.println("String box: " + str);

        // Box for Integer
        Box<Integer> intBox = new Box<>();
        intBox.set(42);
        int value = intBox.get();  // No cast needed
        System.out.println("Integer box: " + value);

        // Box for custom type
        Box<Double> doubleBox = new Box<>();
        doubleBox.set(3.14);
        System.out.println("Double box: " + doubleBox.get());
    }
}
```

**Expected Output:**
```
String box: Hello Generics
Integer box: 42
Double box: 3.14
```

**Step-by-Step Explanation:**

1. `Box<T>` declares a generic class with type parameter `T`.
2. `T` is used as the type for the field `content`, the parameter of `set()`, and the return type of `get()`.
3. `Box<String>` creates a box that can only hold strings.
4. `Box<Integer>` creates a box that can only hold integers.
5. No casts are needed because the compiler knows the type from the type argument.

### Real-World Cases

- **Collections Framework:** `ArrayList<E>`, `HashMap<K,V>`, `LinkedList<E>`
- **Optional:** `Optional<T>` — a container that may or may not hold a value
- **API Responses:** `ResponseEntity<T>` in Spring — wraps a response payload of any type
- **Repository Pattern:** `Repository<T, ID>` — generic data access objects

---

## 2. Multiple Type Parameters

### Definitions

**Core Definition:** A generic class can declare more than one type parameter, allowing it to be parameterized over multiple types simultaneously.

**Technical Definition:** A generic type may be declared with multiple type parameters, separated by commas within the angle brackets. For example, `class Foo<T, S> { … }` declares a class with two type parameters, T and S. The generic `Pair<K, V>` interface is a classic example, with methods `getKey()` returning `K` and `getValue()` returning `V`.

**Beginner-Friendly Explanation:** Multiple type parameters let you say "this class works with two (or more) types." For example, a `Pair<K, V>` holds a key of one type and a value of another type.

### Purposes

- To create containers that hold multiple related values of different types
- To enable type-safe mappings and associations
- To provide flexibility in generic APIs

### Syntax Rules and Structure

```java
class ClassName<T1, T2, ..., Tn> { ... }
```

**Syntax Rules:**

- Type parameters are comma-separated within angle brackets.
- Convention for naming: `T` (Type), `S`, `U`, `V` for additional types; `K` (Key), `V` (Value), `E` (Element), `N` (Number).
- If there are multiple type parameters that denote arbitrary types, one should use letters that neighbor T in the alphabet.

**Constraints and Limitations:**

- Each type parameter is independent and can have its own bounds.
- Type parameters can reference each other in bounds (e.g., `<T, S extends T>`).

### Annotated Code Example

```java
// File: MultipleTypeParamsDemo.java
import java.util.*;

public class MultipleTypeParamsDemo {

    // Generic class with two type parameters
    static class Pair<K, V> {
        private K key;
        private V value;

        public Pair(K key, V value) {
            this.key = key;
            this.value = value;
        }

        public K getKey() { return key; }
        public V getValue() { return value; }

        @Override
        public String toString() {
            return "(" + key + ", " + value + ")";
        }
    }

    // Three type parameters
    static class Triple<A, B, C> {
        private A first;
        private B second;
        private C third;

        public Triple(A first, B second, C third) {
            this.first = first;
            this.second = second;
            this.third = third;
        }

        @Override
        public String toString() {
            return "(" + first + ", " + second + ", " + third + ")";
        }
    }

    public static void main(String[] args) {
        Pair<String, Integer> pair1 = new Pair<>("Age", 30);
        Pair<Integer, String> pair2 = new Pair<>(1, "One");

        System.out.println("Pair 1: " + pair1); // (Age, 30)
        System.out.println("Pair 2: " + pair2); // (1, One)

        Triple<String, Integer, Double> triple =
            new Triple<>("Pi", 3, 3.14);
        System.out.println("Triple: " + triple); // (Pi, 3, 3.14)
    }
}
```

**Expected Output:**
```
Pair 1: (Age, 30)
Pair 2: (1, One)
Triple: (Pi, 3, 3.14)
```

**Step-by-Step Explanation:**

1. `Pair<K, V>` declares two type parameters: `K` for the key and `V` for the value.
2. `Pair<String, Integer>` specifies `K = String` and `V = Integer`.
3. `Pair<Integer, String>` specifies `K = Integer` and `V = String`.
4. `Triple<A, B, C>` demonstrates three type parameters.

### Real-World Cases

- **`Map<K,V>`:** The most common example of multiple type parameters
- **`Map.Entry<K,V>`:** Represents a single key-value pair
- **`BiFunction<T,U,R>`:** A function that takes two arguments and returns a result
- **`Pair<L, R>`:** Used in functional programming and testing frameworks

---

## 3. Generic Fields

### Definitions

**Core Definition:** A generic field is a field declared within a generic class whose type is a type parameter of that class.

**Technical Definition:** Java allows generic fields — a field can be declared with a type parameter as its type. The type parameter must be declared in the class declaration, and the field's type is substituted with the actual type argument when the class is instantiated. Note that generic type parameters cannot be used in static fields, because static members are shared across all instantiations and the type parameter is not available in a static context.

**Beginner-Friendly Explanation:** A generic field is a field that can hold any type, as long as that type is specified when the class is used. For example, in `class Box<T> { private T content; }`, the `content` field can hold a `String` if `T` is `String`, or an `Integer` if `T` is `Integer`.

### Purposes

- To store values of a type that is specified when the generic class is instantiated
- To enable type-safe storage within generic containers
- To avoid casting when retrieving stored values

### Syntax Rules and Structure

```java
class ClassName<T> {
    private T fieldName;  // generic field
}
```

**Syntax Rules:**

- The field's type is the type parameter (`T`, `E`, `K`, `V`, etc.).
- The field can be used in instance methods.
- Static fields cannot use the class's type parameter.
- Generic fields can be declared with bounds if the type parameter has bounds.

**Constraints and Limitations:**

- Static fields cannot be of type parameter type.
- Cannot create arrays of generic field types (`new T[10]` is illegal).

### Annotated Code Example

```java
// File: GenericFieldDemo.java
public class GenericFieldDemo {

    // Generic class with generic fields
    static class Container<T> {
        private T item;           // generic field
        private T backup;         // another generic field

        public Container(T item) {
            this.item = item;
        }

        public T getItem() { return item; }
        public void setItem(T item) { this.item = item; }

        public T getBackup() { return backup; }
        public void setBackup(T backup) { this.backup = backup; }
    }

    public static void main(String[] args) {
        Container<String> stringContainer = new Container<>("Hello");
        stringContainer.setBackup("World");

        System.out.println("Item: " + stringContainer.getItem());   // Hello
        System.out.println("Backup: " + stringContainer.getBackup()); // World

        Container<Integer> intContainer = new Container<>(100);
        intContainer.setBackup(200);

        System.out.println("Item: " + intContainer.getItem());     // 100
        System.out.println("Backup: " + intContainer.getBackup()); // 200
    }
}
```

**Expected Output:**
```
Item: Hello
Backup: World
Item: 100
Backup: 200
```

**Step-by-Step Explanation:**

1. `Container<T>` declares a generic class with type parameter `T`.
2. `item` and `backup` are fields of type `T`.
3. When `Container<String>` is created, `T` becomes `String`, so both fields are `String`.
4. When `Container<Integer>` is created, `T` becomes `Integer`.

### Real-World Cases

- **`Box<T>`:** A container with a generic `content` field
- **`Optional<T>`:** A container with a generic `value` field
- **`ThreadLocal<T>`:** A field that holds a value specific to each thread
- **`AtomicReference<T>`:** A field that holds an atomic reference to a generic value

---

## 4. Generic Methods

### Definitions

**Core Definition:** A generic method is a method that declares its own type parameters, independent of any class-level type parameters.

**Technical Definition:** Type parameters can be declared within method signatures to create generic methods. This is similar to declaring a generic type, but the type parameter's scope is limited to the method in which it's declared. The type parameter list precedes the return type. Generic methods can be defined within non-generic classes, and can be either static or non-static.

**Beginner-Friendly Explanation:** A generic method is a method that works with any type, but the type is specified when you call the method, not when you create the class. For example, a `swap` method that swaps two elements works for any type.

### Purposes

- To write utility methods that work with any type
- To provide type-safe operations without requiring a generic class
- To enable flexible algorithms that adapt to the types they receive
- To reduce code duplication across similar operations

### Syntax Rules and Structure

```java
// Generic method syntax
<T> returnType methodName(T param) { ... }

// Multiple type parameters
<T, U> returnType methodName(T param1, U param2) { ... }

// Bounded type parameter
<T extends Number> returnType methodName(T param) { ... }
```

**Syntax Rules:**

- The type parameter is declared before the return type in angle brackets.
- Type parameters can be used for parameters, return types, and local variables.
- The compiler infers the type argument from the call context.
- A type witness can be used to explicitly specify the type: `ClassName.<Type>methodName(args)`.

**Constraints and Limitations:**

- Type parameters in generic methods are independent of class-level type parameters.
- Cannot be used with primitive types.
- Cannot create arrays of generic type parameters.

### Annotated Code Example

```java
// File: GenericMethodDemo.java
import java.util.*;

public class GenericMethodDemo {

    // Generic method: works with any type
    public static <T> void printArray(T[] array) {
        for (T element : array) {
            System.out.print(element + " ");
        }
        System.out.println();
    }

    // Generic method with bound: T must be Comparable
    public static <T extends Comparable<T>> T findMax(List<T> list) {
        if (list.isEmpty()) {
            throw new IllegalArgumentException("List cannot be empty");
        }
        T max = list.get(0);
        for (T element : list) {
            if (element.compareTo(max) > 0) {
                max = element;
            }
        }
        return max;
    }

    // Generic method with multiple type parameters
    public static <K, V> void printPair(K key, V value) {
        System.out.println(key + " -> " + value);
    }

    public static void main(String[] args) {
        Integer[] intArray = {1, 2, 3, 4, 5};
        String[] strArray = {"A", "B", "C"};

        printArray(intArray);  // 1 2 3 4 5
        printArray(strArray);  // A B C

        List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9);
        System.out.println("Max: " + findMax(numbers)); // 9

        printPair("Age", 30);    // Age -> 30
        printPair(1, "One");     // 1 -> One
    }
}
```

**Expected Output:**
```
1 2 3 4 5
A B C
Max: 9
Age -> 30
1 -> One
```

**Step-by-Step Explanation:**

1. `printArray` is a generic method with type parameter `T`.
2. `findMax` uses a bounded type parameter `<T extends Comparable<T>>`.
3. `printPair` has two type parameters `<K, V>`.
4. The compiler infers the type arguments from the method arguments.

### Real-World Cases

- **`Collections.sort()`:** Generic sorting method
- **`Collections.addAll()`:** Generic bulk add
- **`Arrays.asList()`:** Generic array-to-list conversion
- **`Stream.map()`:** Generic transformation

---

## 5. Generic Constructors

### Definitions

**Core Definition:** A generic constructor is a constructor that declares its own type parameters, independent of any class-level type parameters.

**Technical Definition:** Type parameters can also be declared within constructor signatures to create generic constructors. This is similar to generic methods, but the type parameter's scope is limited to the constructor in which it's declared. Constructors can be generic even if their classes aren't, in which case the type parameter list precedes the constructor name.

**Beginner-Friendly Explanation:** A generic constructor is a constructor that can accept any type, with the type specified when the constructor is called. It is useful for initializing generic fields or performing type-specific operations during object construction.

### Purposes

- To allow constructors to accept type parameters
- To facilitate the creation of objects with generic types
- To enable type-safe initialization of generic fields

### Syntax Rules and Structure

```java
class ClassName {
    <T> ClassName(T param) {
        // constructor body
    }
}
```

**Syntax Rules:**

- The type parameter is declared before the constructor name.
- The type parameter's scope is limited to the constructor.
- Generic constructors can be used in both generic and non-generic classes.

**Constraints and Limitations:**

- The type parameter is not available outside the constructor.
- Cannot create instances of type parameters inside the constructor.

### Annotated Code Example

```java
// File: GenericConstructorDemo.java
public class GenericConstructorDemo {

    // Non-generic class with a generic constructor
    static class Wrapper {
        private Object value;
        private String typeName;

        // Generic constructor: type parameter T
        public <T> Wrapper(T value) {
            this.value = value;
            this.typeName = value.getClass().getSimpleName();
        }

        public Object getValue() { return value; }
        public String getTypeName() { return typeName; }

        @Override
        public String toString() {
            return "Wrapper[" + typeName + ": " + value + "]";
        }
    }

    public static void main(String[] args) {
        Wrapper stringWrapper = new Wrapper("Hello");
        System.out.println(stringWrapper);  // Wrapper[String: Hello]

        Wrapper intWrapper = new Wrapper(42);
        System.out.println(intWrapper);     // Wrapper[Integer: 42]

        Wrapper doubleWrapper = new Wrapper(3.14);
        System.out.println(doubleWrapper);   // Wrapper[Double: 3.14]
    }
}
```

**Expected Output:**
```
Wrapper[String: Hello]
Wrapper[Integer: 42]
Wrapper[Double: 3.14]
```

**Step-by-Step Explanation:**

1. `Wrapper` is a non-generic class.
2. The constructor `<T> Wrapper(T value)` declares its own type parameter `T`.
3. When `new Wrapper("Hello")` is called, `T` is inferred as `String`.
4. The constructor stores the value and records its type name.

### Real-World Cases

- **`EnumSet.copyOf()`:** Generic constructor for creating enum sets
- **`List.of()`:** Generic factory methods that internally use generic constructors
- **Builder Pattern:** Generic constructors for type-safe builders
- **Repository Pattern:** Generic constructors for initializing repositories

---

## 6. Generic Inheritance

### Definitions

**Core Definition:** Generic inheritance is the ability to extend or implement generic classes and interfaces, with type parameters flowing through the inheritance hierarchy.

**Technical Definition:** You can subtype a generic class or interface by extending or implementing it. The relationship between the type parameters of one class or interface and the type parameters of another are determined by the `extends` and `implements` clauses. `ArrayList<E>` implements `List<E>`, and `List<E>` extends `Collection<E>`. So `ArrayList<String>` is a subtype of `List<String>`, which is a subtype of `Collection<String>`.

**Beginner-Friendly Explanation:** Generic inheritance means that when you extend a generic class, you can pass the type parameter along to the subclass, or you can fix it to a specific type.

### Purposes

- To create type-safe inheritance hierarchies
- To preserve type information through the inheritance chain
- To enable specialized generic classes that extend general ones
- To support the Open-Closed Principle in generic designs

### Syntax Rules and Structure

**Four inheritance scenarios:**

```java
// Scenario 1: Generic class extends generic class, preserving T
class Subclass<T> extends GenericSuperclass<T> { ... }

// Scenario 2: Generic class extends generic class with fixed type
class Subclass extends GenericSuperclass<String> { ... }

// Scenario 3: Generic class extends non-generic class
class Subclass<T> extends NonGenericSuperclass { ... }

// Scenario 4: Non-generic class extends generic class with fixed type
class Subclass extends GenericSuperclass<Integer> { ... }
```

**Syntax Rules:**

- If a generic class extends a generic superclass, the subclass must specify the type arguments for the superclass, or declare its own type parameters.
- If the subclass omits the type argument, the type parameter defaults to `Object`.
- The subtype relationship is preserved only if the type argument is not varied.

**Constraints and Limitations:**

- `Box<Integer>` is not a subtype of `Box<Number>`, even though `Integer` is a subtype of `Number`.
- Generic type parameters cannot be used as superclass type arguments in some contexts.

### Annotated Code Example

```java
// File: GenericInheritanceDemo.java
import java.util.*;

public class GenericInheritanceDemo {

    // Generic superclass
    static class Container<T> {
        protected T item;

        public void set(T item) { this.item = item; }
        public T get() { return item; }
    }

    // Scenario 1: Generic subclass preserves T
    static class AdvancedContainer<T> extends Container<T> {
        public void printType() {
            System.out.println("Item type: " + (item != null ?
                item.getClass().getSimpleName() : "null"));
        }
    }

    // Scenario 2: Subclass fixes T to String
    static class StringContainer extends Container<String> {
        public int length() {
            return get().length();
        }
    }

    // Scenario 4: Non-generic class extends generic class
    static class IntegerContainer extends Container<Integer> {
        public int doubleValue() {
            return get() * 2;
        }
    }

    public static void main(String[] args) {
        // Scenario 1: AdvancedContainer<String>
        AdvancedContainer<String> adv = new AdvancedContainer<>();
        adv.set("Hello");
        System.out.println("Advanced: " + adv.get()); // Hello
        adv.printType(); // Item type: String

        // Scenario 2: StringContainer (T fixed to String)
        StringContainer strContainer = new StringContainer();
        strContainer.set("Hello World");
        System.out.println("Length: " + strContainer.length()); // 11

        // Scenario 4: IntegerContainer (T fixed to Integer)
        IntegerContainer intContainer = new IntegerContainer();
        intContainer.set(21);
        System.out.println("Double: " + intContainer.doubleValue()); // 42
    }
}
```

**Expected Output:**
```
Advanced: Hello
Item type: String
Length: 11
Double: 42
```

**Step-by-Step Explanation:**

1. `Container<T>` is the generic superclass.
2. `AdvancedContainer<T>` preserves `T` through inheritance.
3. `StringContainer` fixes `T` to `String`, so `get()` returns `String`.
4. `IntegerContainer` fixes `T` to `Integer`, so `get()` returns `Integer`.

### Real-World Cases

- **Collections Framework:** `ArrayList<E>` extends `AbstractList<E>` implements `List<E>`
- **Stream API:** `Stream<T>` and `IntStream` — specialized subinterfaces
- **Repository Pattern:** `JpaRepository<T, ID>` extends `PagingAndSortingRepository<T, ID>`
- **Builder Pattern:** `GenericBuilder<T>` extended by specific builders

---

## 7. Generic Best Practices

### Definitions

**Core Definition:** Best practices for generic classes are guidelines for writing type-safe, flexible, and maintainable generic code.

**Technical Definition:** Best practices derive from *Effective Java* and Oracle's Java Tutorials. Key rules include: do not use raw types; eliminate unchecked warnings; prefer lists to arrays; favor generic types for reusable components; and use bounded wildcards to increase API flexibility.

**Beginner-Friendly Explanation:** Best practices help you write generic code that is safe, clear, and easy to use.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Do not use raw types** | Always use parameterized types (`List<String>` not `List`) |
| **Eliminate unchecked warnings** | Fix warnings rather than suppressing them; keep `@SuppressWarnings` local and justified |
| **Prefer lists to arrays** | Arrays are covariant and reified; lists are safer with generics |
| **Favor generic types** | Use generic types for reusable components |
| **Favor generic methods** | Use method-level generics when the class doesn't need a type parameter |
| **Use bounded type parameters** | Restrict types when you need to call methods from the bound |
| **Use PECS for wildcards** | `? extends T` for producers, `? super T` for consumers |
| **Use the diamond operator** | `new ArrayList<>()` instead of `new ArrayList<String>()` |
| **Follow naming conventions** | `T` for Type, `E` for Element, `K` for Key, `V` for Value |
| **Document type parameters** | Explain type parameter meanings in Javadoc |
| **Avoid using same names for method and class type parameters** | Prevents confusion |

### Annotated Code Example

```java
// File: GenericBestPractices.java
import java.util.*;

public class GenericBestPractices {

    /**
     * Generic method with bounded type parameter.
     * @param <T> the type of elements, must be Comparable
     */
    public static <T extends Comparable<T>> T max(List<T> list) {
        T max = list.get(0);
        for (T element : list) {
            if (element.compareTo(max) > 0) {
                max = element;
            }
        }
        return max;
    }

    // GOOD: PECS - producer extends (reading from collection)
    public static double sum(List<? extends Number> numbers) {
        double total = 0;
        for (Number n : numbers) {
            total += n.doubleValue();
        }
        return total;
    }

    // GOOD: PECS - consumer super (writing to collection)
    public static void addNumbers(List<? super Integer> list) {
        for (int i = 1; i <= 5; i++) {
            list.add(i);
        }
    }

    // GOOD: Generic method with multiple type parameters
    public static <K, V> Map<K, V> createMap(K key, V value) {
        Map<K, V> map = new HashMap<>();
        map.put(key, value);
        return map;
    }

    public static void main(String[] args) {
        // GOOD: Diamond operator
        List<Integer> numbers = new ArrayList<>();
        numbers.addAll(Arrays.asList(5, 2, 8, 1, 9));

        System.out.println("Max: " + max(numbers)); // 9

        // GOOD: PECS in action
        List<Integer> ints = Arrays.asList(1, 2, 3);
        List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);
        System.out.println("Sum ints: " + sum(ints));     // 6.0
        System.out.println("Sum doubles: " + sum(doubles)); // 6.6

        List<Number> numberList = new ArrayList<>();
        addNumbers(numberList);
        System.out.println("Numbers: " + numberList); // [1, 2, 3, 4, 5]

        // GOOD: Generic method inference
        Map<String, Integer> map = createMap("Age", 30);
        System.out.println("Map: " + map); // {Age=30}
    }
}
```

**Expected Output:**
```
Max: 9
Sum ints: 6.0
Sum doubles: 6.6
Numbers: [1, 2, 3, 4, 5]
Map: {Age=30}
```

**Step-by-Step Explanation:**

1. `max` uses a bounded type parameter `<T extends Comparable<T>>`.
2. `sum` uses PECS: `List<? extends Number>` for reading.
3. `addNumbers` uses PECS: `List<? super Integer>` for writing.
4. `createMap` demonstrates a generic method with multiple type parameters.
5. The diamond operator is used for type inference.

### Real-World Cases

- **Collections Framework:** Uses all best practices extensively
- **Spring Framework:** `ResponseEntity<T>`, `Optional<T>`, and other generic types
- **Guava:** `ImmutableList<E>`, `Optional<T>`, `Multimap<K,V>`
- **JPA:** `JpaRepository<T, ID>`, `TypedQuery<X>`

---

## References Links

- Generic Types — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/types.html
- Generic Methods and Constructors — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/methods.html
- Bounded Type Parameters — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/bounded.html
- Generics, Inheritance, and Subtypes — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/generics/inheritance.html
- Generic Methods — Dev.java - https://dev.java/learn/generics/methods/
- Java Generics Best Practices — Plinth (GitHub) - https://github.com/jabrena/plinth/blob/cbec11f42fe50677e50919384da233678a0c74fe/skills/128-java-generics/SKILL.md
- Effective Java (3rd Edition) by Joshua Bloch — Items 26-29: Generics
- Java Generics Explained — Cleverence - https://www.cleverence.com/articles/oracle-documentation/generic-types-java-tutorials-4729/
- Declaring Generic Types — Eclipse AspectJ Documentation - https://eclipse.dev/aspectj/doc/latest/adk15notebook/generics.html