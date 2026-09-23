# Java Composition: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Composition** is a design technique in Java where a class contains an instance of another class as a field, establishing a **"Has-A" relationship** rather than an "Is-A" relationship. The containing class delegates work to the contained object, enabling code reuse without the tight coupling of inheritance.

### Technical Definition

Composition is a form of association where one class (the *composite* or *wrapper*) holds a reference to an instance of another class (the *component* or *delegate*). The composite implements its behavior by **forwarding** method calls to the contained instance, a technique also known as **delegation**. Unlike inheritance, composition does not expose the internal implementation of the delegate; the abstraction barrier between the two classes is preserved.

### Beginner-Friendly Explanation

Think of a car. A car **has** an engine, a car **has** wheels, a car **has** a stereo. You wouldn't say "a car *is* an engine." Instead, the car contains these parts and uses them. If you want to build a different car, you can swap in a different engine without redesigning the whole car. That's composition: building objects out of other objects.

### Key Characteristics

- **Has-A relationship**: The class contains an instance of another class as a field.
- **Forwarding/Delegation**: Method calls are passed to the contained object.
- **Encapsulation preserved**: The delegate's internal details are hidden from the composite's clients.
- **Runtime flexibility**: The delegate can be swapped at runtime (e.g., via constructor injection).
- **No inheritance coupling**: Changes to the delegate's implementation don't break the composite.
- **Multiple reuse**: A class can compose many different objects, unlike single inheritance.

### Prerequisites

- Basic understanding of classes, fields, and methods.
- Familiarity with interfaces (used to type the delegate).
- Knowledge of inheritance and its trade-offs.

### Related Programming Areas

- **Design Patterns**: Decorator, Strategy, and Wrapper patterns rely on composition.
- **Dependency Injection**: Composition is the foundation of DI frameworks.
- **API Design**: Composition enables flexible, loosely coupled APIs.
- **Effective Java**: Item 18: "Favor composition over inheritance".

### Core Concepts / Features

1. Has-A Relationships
2. Delegation (Forwarding Method Calls)
3. Composition vs. Inheritance Trade-offs
4. Reusable Object Collaboration


## Core Concept 1: Has-A Relationships

### Definitions

**Core Definition**: A **Has-A relationship** exists when one class contains an instance of another class as a field, meaning the containing object "has" the contained object as a component.

**Technical Definition**: In composition, the instance variable of one class is declared inside another class. This creates a strong association where the composite object owns or uses the component object. The relationship is defined at runtime through object references, not at compile time through inheritance.

**Beginner-Friendly Explanation**: A "Has-A" relationship is like saying "a car has an engine." The car doesn't *become* an engine—it contains one and uses it. You can describe the car by what it has.

### Purposes

- To model real-world relationships where objects are composed of other objects.
- To enable code reuse without the constraints of inheritance.
- To allow flexible, interchangeable components at runtime.
- To keep classes decoupled and independently testable.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
class Composite {
    private Component component;  // Has-A field

    public Composite(Component component) {
        this.component = component;
    }

    // Delegate methods to component
    public ReturnType methodName() {
        return component.methodName();
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Component` | The type of the contained object (often an interface). |
| `component` | The field holding the reference. |
| Constructor | Injects or creates the component. |

**Syntax Rules:**

- The component is declared as a `private` field (encapsulation).
- The component is typically typed as an interface or abstract class for flexibility.
- The composite's methods delegate to the component's methods.
- The composite can add its own behavior before or after delegation.

**Constraints and Limitations:**

- The composite must ensure the component is initialized before use (avoid `NullPointerException`).
- The component's lifetime may be tied to the composite (strong composition) or independent (aggregation).
- Wrapper classes are not suitable for callback frameworks (the "SELF problem").

### Annotated Complete Code Examples

**Example 1: Has-A Relationship with Delegation**

```java
/**
 * Demonstrates a Has-A relationship using composition.
 */
// Component interface
interface Engine {
    void start();
    int getHorsepower();
}

class V8Engine implements Engine {
    public void start() {
        System.out.println("V8 engine roars to life");
    }

    public int getHorsepower() { 
        return 450; 
    }
}

class ElectricEngine implements Engine {
    public void start() {
        System.out.println("Electric engine hums quietly");
    }
    
    public int getHorsepower() { 
        return 300; 
    }
}

// Composite class: Car HAS-A Engine
class Car {
    private final Engine engine;  // Composition field

    public Car(Engine engine) {
        this.engine = engine;
    }

    // Delegation: forward start() to the engine
    public void start() {
        engine.start();
    }

    // Delegation with additional logic
    public void displayHorsepower() {
        System.out.println("Horsepower: " + engine.getHorsepower());
    }
}

public class HasARelationshipDemo {
    public static void main(String[] args) {
        // Car with V8 engine
        Car sportsCar = new Car(new V8Engine());
        sportsCar.start();
        sportsCar.displayHorsepower();

        // Car with electric engine
        Car electricCar = new Car(new ElectricEngine());
        electricCar.start();
        electricCar.displayHorsepower();
    }
}
```

**Expected Output:**

```
V8 engine roars to life
Horsepower: 450
Electric engine hums quietly
Horsepower: 300
```

**Why This Output Occurs:**
- `Car` has a `private final Engine engine` field—a Has-A relationship.
- `start()` and `displayHorsepower()` forward calls to the engine.
- The engine type is determined at runtime, so the same `Car` class works with different engines.

**Step-by-Step Setup Guide:**
1. Create the interface and implementation files.
2. Create the `Car` composite class.
3. Run the demo and observe the output.

### Real-World Cases

- **`HashMap`**: Uses a `Hashtable` internally (composition), not inheritance.
- **`Properties`**: The JDK got this wrong—it extends `Hashtable` when composition would have been better.
- **Vehicle systems**: A `Car` has an `Engine`, `Transmission`, and `Wheels`.
- **Computer systems**: A `Computer` has a `CPU`, `Memory`, and `Storage`.

### References

- Scaler Topics – Has-A Relationship in Java - https://www.scaler.com/topics/has-a-relationship-in-java/
- CMU – Class/Object Relationships - http://public.africa.cmu.edu/cbishop/jsmart/relationships.html


## Core Concept 2: Delegation (Forwarding Method Calls)

### Definitions

**Core Definition**: **Delegation** (also called **forwarding**) is the technique where the composite class implements a method by invoking the corresponding method on the contained object and returning the result.

**Technical Definition**: Composition and forwarding (also called delegation) is a design pattern where a forwarder class contains a private member field of the delegated type and provides wrapper methods that invoke the corresponding methods on the contained instance. The forwarding methods are trivial but essential: they preserve the interface contract while delegating the actual work.

**Beginner-Friendly Explanation**: Delegation is like a manager who says "I'll handle your request" but then immediately passes it to the specialist who actually does the work. The manager is the "face" of the operation, but the specialist does the job.

### Purposes

- To reuse behavior without inheriting implementation details.
- To preserve encapsulation by hiding the delegate's internal structure.
- To enable runtime flexibility (swap delegates without changing the composite).
- To implement the Wrapper/Decorator pattern for adding behavior.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class Composite implements ComponentInterface {
    private final ComponentInterface delegate;

    public Composite(ComponentInterface delegate) {
        this.delegate = delegate;
    }

    @Override
    public ReturnType methodName(Parameters params) {
        // Optional: pre-processing
        ReturnType result = delegate.methodName(params);
        // Optional: post-processing
        return result;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `delegate` | The contained object to which calls are forwarded. |
| `methodName` | The forwarded method. |
| Pre/post processing | Optional logic around the delegation. |

**Syntax Rules:**

- The composite implements the same interface as the delegate.
- Each method delegates to the corresponding method on the delegate.
- The delegate is typically `final` to ensure it's set once.
- Pre-processing can validate or log before delegating.
- Post-processing can modify the result or perform cleanup.

**Constraints and Limitations:**

- Writing forwarding methods is tedious but necessary (no automatic delegation in Java).
- The "SELF problem": if the delegate calls back to itself, it bypasses the wrapper.
- Delegation is not the same as inheritance; `super` calls have no equivalent in composition.

### Annotated Complete Code Examples

**Example 1: Forwarding with Instrumentation**

```java
import java.util.*;

/**
 * Demonstrates composition and forwarding with instrumentation.
 */
class InstrumentedSet<E> implements Set<E> {
    private final Set<E> delegate;
    private int addCount = 0;

    public InstrumentedSet(Set<E> delegate) {
        this.delegate = delegate;
    }

    @Override
    public boolean add(E element) {
        addCount++;  // Pre-processing: instrument the call
        return delegate.add(element);  // Forward to delegate
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return delegate.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    // Forward remaining methods to the delegate
    @Override public int size() { return delegate.size(); }
    @Override public boolean isEmpty() { return delegate.isEmpty(); }
    @Override public boolean contains(Object o) { return delegate.contains(o); }
    @Override public Iterator<E> iterator() { return delegate.iterator(); }
    @Override public Object[] toArray() { return delegate.toArray(); }
    @Override public <T> T[] toArray(T[] a) { return delegate.toArray(a); }
    @Override public boolean remove(Object o) { return delegate.remove(o); }
    @Override public boolean containsAll(Collection<?> c) { return delegate.containsAll(c); }
    @Override public boolean retainAll(Collection<?> c) { return delegate.retainAll(c); }
    @Override public boolean removeAll(Collection<?> c) { return delegate.removeAll(c); }
    @Override public void clear() { delegate.clear(); }

    public static void main(String[] args) {
        Set<String> instrumented = new InstrumentedSet<>(new HashSet<>());
        instrumented.add("Java");
        instrumented.add("Composition");
        instrumented.add("Delegation");

        System.out.println("Set contents: " + instrumented);
        System.out.println("Add count: " + ((InstrumentedSet<String>) instrumented).getAddCount());
    }
}
```

**Expected Output:**

```
Set contents: [Java, Composition, Delegation]
Add count: 3
```

**Why This Output Occurs:**
- `InstrumentedSet` wraps a `HashSet` (the delegate).
- Each `add()` call increments `addCount` and forwards to the delegate.
- The wrapper adds instrumentation without modifying `HashSet`.

### Real-World Cases

- **`Collections.synchronizedSet()`**: Wraps a set for thread safety.
- **`Collections.unmodifiableList()`**: Wraps a list to prevent modification.
- **Google Guava**: Provides forwarding classes for all collection interfaces.
- **Decorator Pattern**: Adds behavior to objects dynamically.

### References

- CMU SEI CERT – Composition and Forwarding - https://wiki.sei.cmu.edu/confluence/pages/diffpages.action?originalId=88881913&pageId=88871798
- Oracle Java Magazine – Favor Composition Over Inheritance - https://blogs.oracle.com/content/published/api/v1.1/assets/CONT2E5A3210DD844F7893E5199CD3DB0D83/native/You+should+favor+composition+over+inheritance+in+Java.+Here%E2%80%99s+why_.pdf


## Core Concept 3: Composition vs. Inheritance Trade-offs

### Definitions

**Core Definition**: **Composition** uses a Has-A relationship (containing objects); **inheritance** uses an Is-A relationship (extending classes). Each has distinct trade-offs in coupling, flexibility, and encapsulation.

**Technical Definition**: Inheritance creates tight coupling because the subclass depends on the superclass's implementation details. Changes to the superclass can break subclasses even if the subclass code is unchanged. Composition avoids this by decoupling the composite from the delegate's implementation; the delegate can be swapped or modified independently.

**Beginner-Friendly Explanation**: Inheritance is like being born into a family—you inherit traits whether you want them or not, and you're stuck with them. Composition is like choosing your friends—you can pick who you want to work with, and you can change your mind later.

### Purposes

- To choose the right reuse mechanism for the problem.
- To understand the risks of inheritance and when to avoid it.
- To design flexible, maintainable systems.
- To apply the "Favor composition over inheritance" principle effectively.

### Syntax Rules and Structure

**Comparison Table:**

| Aspect | Inheritance | Composition |
|--------|-------------|-------------|
| Relationship | Is-A | Has-A |
| Coupling | Tight | Loose |
| Encapsulation | Breaks (protected members exposed) | Preserved |
| Flexibility | Compile-time | Runtime |
| Reuse scope | Single class | Multiple objects |
| Method reuse | Automatic | Manual forwarding |
| Polymorphism | Yes (subtype) | Yes (interface) |

**When to Use Inheritance:**

- The subclass is genuinely a subtype of the superclass (Liskov Substitution Principle).
- The superclass is designed and documented for extension.
- You need polymorphic substitutability.

**When to Use Composition:**

- You only need some of the behavior (not the full type).
- The delegate might change at runtime.
- The delegate's implementation might change independently.
- You want to avoid the fragile base class problem.

**Constraints and Limitations:**

- Inheritance is simpler to write (one keyword vs. many forwarding methods).
- Composition requires more boilerplate but is safer.
- Inheritance can be appropriate for framework extension points.

### Annotated Complete Code Examples

**Example 1: Inheritance Problem and Composition Solution**

```java
import java.util.*;

/**
 * Demonstrates why composition is preferred over inheritance.
 */
// BROKEN: Inheritance breaks when superclass adds new methods
class BrokenInstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() { return addCount; }
}

// FIXED: Composition with forwarding
class FixedInstrumentedSet<E> implements Set<E> {
    private final Set<E> delegate;
    private int addCount = 0;

    public FixedInstrumentedSet(Set<E> delegate) {
        this.delegate = delegate;
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return delegate.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return delegate.addAll(c);
    }

    public int getAddCount() { return addCount; }

    // Forward all other methods...
    @Override public int size() { return delegate.size(); }
    @Override public boolean isEmpty() { return delegate.isEmpty(); }
    @Override public boolean contains(Object o) { return delegate.contains(o); }
    @Override public Iterator<E> iterator() { return delegate.iterator(); }
    @Override public Object[] toArray() { return delegate.toArray(); }
    @Override public <T> T[] toArray(T[] a) { return delegate.toArray(a); }
    @Override public boolean remove(Object o) { return delegate.remove(o); }
    @Override public boolean containsAll(Collection<?> c) { return delegate.containsAll(c); }
    @Override public boolean retainAll(Collection<?> c) { return delegate.retainAll(c); }
    @Override public boolean removeAll(Collection<?> c) { return delegate.removeAll(c); }
    @Override public void clear() { delegate.clear(); }
}

public class CompositionVsInheritanceDemo {
    public static void main(String[] args) {
        // Inheritance version: addAll calls add internally, double-counting
        BrokenInstrumentedHashSet<String> broken = new BrokenInstrumentedHashSet<>();
        broken.addAll(Arrays.asList("a", "b", "c"));
        System.out.println("Broken count (should be 3): " + broken.getAddCount());

        // Composition version: addAll forwards directly, no double-counting
        FixedInstrumentedSet<String> fixed = new FixedInstrumentedSet<>(new HashSet<>());
        fixed.addAll(Arrays.asList("a", "b", "c"));
        System.out.println("Fixed count (should be 3): " + fixed.getAddCount());
    }
}
```

**Expected Output:**

```
Broken count (should be 3): 6
Fixed count (should be 3): 3
```

**Why This Output Occurs:**
- `HashSet.addAll()` internally calls `add()` for each element.
- The inherited `addAll()` calls `super.addAll()`, which calls the overridden `add()`, double-counting.
- Composition forwards `addAll()` directly to the delegate without triggering the wrapper's `add()`.

### Real-World Cases

- **`Properties extends Hashtable`**: The JDK made the wrong choice; composition would have been better.
- **Security frameworks**: Composition avoids security holes from unoverridden methods.
- **Collection wrappers**: `Collections.synchronizedSet()` uses composition.

### References

- MIT – Reading 14: Inheritance & Composition - https://web.mit.edu/6.005/www/fa14/classes/14-inheritance/
- CMU – Principles of Software Construction - http://www.cs.cmu.edu/~mhilton/classes/17-214/s19/slides/20190205-design-patterns-for-reuse.pdf
- Oracle Java Magazine – Favor Composition Over Inheritance - https://blogs.oracle.com/content/published/api/v1.1/assets/CONT2E5A3210DD844F7893E5199CD3DB0D83/native/You+should+favor+composition+over+inheritance+in+Java.+Here%E2%80%99s+why_.pdf


## Core Concept 4: Reusable Object Collaboration

### Definitions

**Core Definition**: **Reusable object collaboration** is the practice of building flexible systems by composing objects that can be swapped, extended, or replaced without modifying existing code.

**Technical Definition**: Composition enables the **Wrapper** (or **Decorator**) pattern, where a class implements an interface by forwarding calls to a contained instance of that interface. This allows the wrapper to add behavior (instrumentation, synchronization, logging) while remaining compatible with any implementation of the interface. The design is enabled by interface-based programming: the delegate is typed as an interface, not a concrete class.

**Beginner-Friendly Explanation**: Reusable object collaboration is like LEGO bricks. Each brick (object) has a standard connector (interface), so you can combine them in many ways. You can put a "logging brick" in front of a "database brick," or a "caching brick" in front of a "network brick," and they all fit together.

### Purposes

- To add behavior to objects without modifying their classes.
- To enable runtime flexibility in object composition.
- To create reusable components that work with any implementation.
- To support the Open/Closed Principle (open for extension, closed for modification).

### Syntax Rules and Structure

**Complete General Syntax (Wrapper Pattern):**

```java
public class Wrapper<T> implements Interface<T> {
    private final Interface<T> wrapped;

    public Wrapper(Interface<T> wrapped) {
        this.wrapped = wrapped;
    }

    @Override
    public ReturnType method(Parameters p) {
        // additional behavior
        return wrapped.method(p);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Interface` | The contract implemented by both wrapper and delegate. |
| `wrapped` | The contained delegate. |
| Override methods | Forward to delegate with optional enhancement. |

**Syntax Rules:**

- The wrapper implements the same interface as the wrapped object.
- The wrapper accepts any implementation of the interface.
- The wrapper can be layered (wrapper wrapping a wrapper).
- The wrapper's constructor takes the interface type, not a concrete class.

**Constraints and Limitations:**

- Forwarding all interface methods is tedious (no automatic delegation in Java).
- The SELF problem: callbacks bypass the wrapper.
- Some performance overhead from additional method calls (usually negligible).

### Annotated Complete Code Examples

**Example 1: Layered Wrappers for Logging and Caching**

```java
import java.util.*;

/**
 * Demonstrates reusable object collaboration with layered wrappers.
 */
interface DataSource {
    String fetch(String key);
}

class DatabaseSource implements DataSource {
    public String fetch(String key) {
        System.out.println("Fetching from database: " + key);
        return "Data for " + key;
    }
}

// Wrapper 1: Adds logging
class LoggingDataSource implements DataSource {
    private final DataSource delegate;

    public LoggingDataSource(DataSource delegate) {
        this.delegate = delegate;
    }

    public String fetch(String key) {
        System.out.println("[LOG] Fetching key: " + key);
        String result = delegate.fetch(key);
        System.out.println("[LOG] Result: " + result);
        return result;
    }
}

// Wrapper 2: Adds caching
class CachingDataSource implements DataSource {
    private final DataSource delegate;
    private final Map<String, String> cache = new HashMap<>();

    public CachingDataSource(DataSource delegate) {
        this.delegate = delegate;
    }

    public String fetch(String key) {
        if (cache.containsKey(key)) {
            System.out.println("[CACHE] Hit for: " + key);
            return cache.get(key);
        }
        String result = delegate.fetch(key);
        cache.put(key, result);
        return result;
    }
}

public class LayeredWrapperDemo {
    public static void main(String[] args) {
        // Layer 1: Database
        DataSource source = new DatabaseSource();

        // Layer 2: Caching
        source = new CachingDataSource(source);

        // Layer 3: Logging
        source = new LoggingDataSource(source);

        // First fetch: goes through all layers
        source.fetch("user:1");

        // Second fetch: cache hit
        source.fetch("user:1");
    }
}
```

**Expected Output:**

```
[LOG] Fetching key: user:1
Fetching from database: user:1
[LOG] Result: Data for user:1
[LOG] Fetching key: user:1
[CACHE] Hit for: user:1
[LOG] Result: Data for user:1
```

**Why This Output Occurs:**
- `LoggingDataSource` wraps `CachingDataSource`, which wraps `DatabaseSource`.
- First fetch: logging → cache miss → database → return through layers.
- Second fetch: logging → cache hit → return (no database call).

### Real-World Cases

- **`Collections.synchronizedList()`**: Wraps a list for thread safety.
- **Java I/O Streams**: `BufferedInputStream` wraps `FileInputStream`.
- **Google Guava**: `ForwardingSet`, `ForwardingList`, etc.
- **Spring AOP**: Proxies wrap beans for logging, security, transactions.

### References

- MIT – Reading 14: Inheritance & Composition - https://web.mit.edu/6.005/www/fa14/classes/14-inheritance/
- CMU – Principles of Software Construction - http://www.cs.cmu.edu/~mhilton/classes/17-214/s19/slides/20190205-design-patterns-for-reuse.pdf


## References

- CMU – Class/Object Relationships - http://public.africa.cmu.edu/cbishop/jsmart/relationships.html
- CMU SEI CERT – Composition and Forwarding - https://wiki.sei.cmu.edu/confluence/pages/diffpages.action?originalId=88881913&pageId=88871798
- CMU – Principles of Software Construction - http://www.cs.cmu.edu/~mhilton/classes/17-214/s19/slides/20190205-design-patterns-for-reuse.pdf
- CMU SEI CERT – Composition and Forwarding (Calendar) - https://wiki.sei.cmu.edu/confluence/pages/diffpages.action?originalId=88491868&pageId=88491766
- MIT – Reading 14: Inheritance & Composition - https://web.mit.edu/6.005/www/fa14/classes/14-inheritance/
- Oracle Java Magazine – Favor Composition Over Inheritance - https://blogs.oracle.com/content/published/api/v1.1/assets/CONT2E5A3210DD844F7893E5199CD3DB0D83/native/You+should+favor+composition+over+inheritance+in+Java.+Here%E2%80%99s+why_.pdf
- Scaler Topics – Has-A Relationship in Java - https://www.scaler.com/topics/has-a-relationship-in-java/
- OpenJDK – Downsizing the delegate/composition pattern ceremony - https://mail.openjdk.org/pipermail/amber-dev/2020-August/006427.html