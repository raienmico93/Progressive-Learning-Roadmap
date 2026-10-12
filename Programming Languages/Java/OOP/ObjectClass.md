# Java Object Class and Objects Utility Class: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `Object` class is the root of the Java class hierarchy, meaning every class in Java either directly or indirectly extends `Object`. It provides fundamental methods that all objects inherit, including `equals()`, `hashCode()`, `toString()`, `getClass()`, `clone()`, and the thread coordination methods `wait()`, `notify()`, and `notifyAll()`. The `Objects` utility class, introduced in Java 7, is a final class consisting of static utility methods for operating on objects, including null-safe or null-tolerant methods for computing hash codes, returning strings, comparing objects, and checking index bounds.

**Technical Definition:** "The class `Object` is the single root of the class hierarchy. All objects, including arrays, implement the methods of this class". "Class `Object` is the root of the class hierarchy. Every class has `Object` as a superclass. All objects, including arrays, implement the methods of this class". The `Objects` class "consists of `static` utility methods for operating on objects, or checking certain conditions before operation. These utilities include `null`-safe or `null`-tolerant methods for computing the hash code of an object, returning a string for an object, comparing two objects, and checking if indexes or sub-range values are out of bounds".

**Beginner-Friendly Explanation:** Every Java object, no matter what kind, is automatically a child of the `Object` class. This means every object comes with a standard set of built-in methods: one to compare itself to another object (`equals`), one to generate a unique-ish number for it (`hashCode`), one to describe itself as text (`toString`), and a few others. The `Objects` utility class is like a toolbox filled with helper methods that make working with these objects — especially when they might be `null` — much easier and safer.

### Key Characteristics

- **Universal Superclass:** `Object` is the root of the class hierarchy; every class inherits from it directly or indirectly.
- **Default Behaviour:** `Object` provides default implementations for `equals()`, `hashCode()`, `toString()`, and `clone()` that subclasses often override.
- **Thread Coordination:** `wait()`, `notify()`, and `notifyAll()` are the foundation of Java's intrinsic locking and inter-thread communication.
- **Final Methods:** `getClass()`, `wait()`, `notify()`, and `notifyAll()` are `final` and cannot be overridden.
- **Deprecated Methods:** `finalize()` is deprecated for removal as of JDK 18 (JEP 421).
- **Null Safety:** The `Objects` utility class provides null-safe and null-tolerant methods to simplify common operations and reduce `NullPointerException` risks.

### Prerequisites

- Basic understanding of Java classes and objects
- Familiarity with method overriding and the `@Override` annotation
- Knowledge of hash-based collections (`HashMap`, `HashSet`)
- Awareness of threading concepts (for `wait`/`notify`)

### Related Programming Areas

- **Collections Framework:** `equals()` and `hashCode()` are critical for correct behaviour in `HashMap`, `HashSet`, and other hash-based collections.
- **Concurrency:** `wait()`, `notify()`, and `notifyAll()` are fundamental to inter-thread communication.
- **Serialization:** `Serializable` interacts with `Object`'s cloning and serialization mechanisms.
- **Reflection:** `getClass()` is the entry point for Java's reflection API.
- **Utility Programming:** The `Objects` class is widely used in modern Java for null-safe operations and validation.

### Core Concepts / Features

---

## 1. Object class

### Definitions

**Core Definition:** The `Object` class is the root of the Java class hierarchy from which all other classes inherit.

**Technical Definition:** `public class Object` is the single root of the class hierarchy. "All objects, including arrays, implement the methods of this class". It was introduced in JDK 1.0 and is declared as `public class Object` with a no-argument constructor `Object()`.

**Beginner-Friendly Explanation:** `Object` is like the "great-grandparent" of every class in Java. Everything you create in Java is, at some level, an `Object`. This means every object you make automatically has methods like `toString()`, `equals()`, and `hashCode()`, even if you didn't write them yourself.

### Purposes

- To serve as the common root of the class hierarchy, enabling polymorphic treatment of all objects
- To provide default implementations of fundamental methods that all objects need
- To define the contract for object equality, hashing, and string representation
- To support thread coordination through `wait()`, `notify()`, and `notifyAll()`
- To enable runtime type inspection through `getClass()`

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class Object {
    public Object() { }
    public final Class<?> getClass() { ... }
    public int hashCode() { ... }
    public boolean equals(Object obj) { ... }
    protected Object clone() throws CloneNotSupportedException { ... }
    public String toString() { ... }
    public final void notify() { ... }
    public final void notifyAll() { ... }
    public final void wait() throws InterruptedException { ... }
    public final void wait(long timeout) throws InterruptedException { ... }
    public final void wait(long timeout, int nanos) throws InterruptedException { ... }
    @Deprecated(since="9", forRemoval=true)
    protected void finalize() throws Throwable { ... }
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `getClass()` | Returns the runtime class of this object. |
| `hashCode()` | Returns a hash code value for the object. |
| `equals(Object obj)` | Indicates whether some other object is "equal to" this one. |
| `clone()` | Creates and returns a copy of this object. |
| `toString()` | Returns a string representation of the object. |
| `notify()` | Wakes up a single thread waiting on this object's monitor. |
| `notifyAll()` | Wakes up all threads waiting on this object's monitor. |
| `wait()` | Causes the current thread to wait until notified. |
| `finalize()` | **Deprecated.** Called by the garbage collector before object is collected. |

**Syntax Rules:**

- All classes implicitly extend `Object`.
- The `Object` class is in the `java.lang` package and is automatically imported.
- Methods like `getClass()`, `wait()`, `notify()`, and `notifyAll()` are `final` and cannot be overridden.

**Constraints and Limitations:**

- `Object` itself does not implement `Cloneable`, so calling `clone()` on a plain `Object` throws `CloneNotSupportedException`.
- `finalize()` is deprecated for removal and should not be used in new code.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Exploring Object Methods**

```java
// File: ObjectClassDemo.java
public class ObjectClassDemo {
    private String name;

    public ObjectClassDemo(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "ObjectClassDemo{name='" + name + "'}";
    }

    public static void main(String[] args) {
        ObjectClassDemo obj = new ObjectClassDemo("Alice");

        // getClass() - runtime class
        System.out.println("Class: " + obj.getClass().getName());

        // toString() - string representation
        System.out.println("toString: " + obj.toString());

        // hashCode() - hash code
        System.out.println("hashCode: " + obj.hashCode());

        // equals() - reference equality (default)
        ObjectClassDemo obj2 = new ObjectClassDemo("Alice");
        System.out.println("equals: " + obj.equals(obj2));
    }
}
```

**Expected Output:**
```
Class: ObjectClassDemo
toString: ObjectClassDemo{name='Alice'}
hashCode: 123456789
equals: false
```

**Step-by-Step Explanation:**

1. `obj.getClass().getName()` returns the fully qualified name of the runtime class.
2. `obj.toString()` uses the overridden `toString()` method.
3. `obj.hashCode()` returns the default hash code (typically based on the memory address).
4. `obj.equals(obj2)` returns `false` because the default `equals()` uses reference equality — `obj` and `obj2` are different objects.

### Real-World Cases

- **Collections:** `HashMap` and `HashSet` rely on `equals()` and `hashCode()` inherited from `Object` (or overridden in subclasses).
- **Logging:** `toString()` is called automatically when objects are logged or printed.
- **Reflection:** `getClass()` is the starting point for introspection.
- **Threading:** `wait()`/`notify()` are used in producer-consumer and other concurrency patterns.

---

## 2. equals()

### Definitions

**Core Definition:** The `equals(Object obj)` method determines whether some other object is "equal to" this one.

**Technical Definition:** "The `equals` method implements an equivalence relation on non-null object references". It must be reflexive, symmetric, transitive, consistent, and return `false` for `null`. The default implementation in `Object` returns `true` only if the argument is the same object (`this == obj`).

**Beginner-Friendly Explanation:** `equals()` answers the question: "Are these two objects the same in terms of content?" By default, it checks if they are literally the same object (like checking if two people are the same person), but you can override it to compare contents (like checking if two people have the same name and ID).

### Purposes

- To define logical equality for a class
- To enable correct behaviour in hash-based collections (`HashMap`, `HashSet`)
- To support value-based comparison of objects
- To implement domain-specific equality rules

### Syntax Rules and Structure

**Complete General Syntax:**

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    ClassName other = (ClassName) obj;
    return field1 == other.field1 && Objects.equals(field2, other.field2);
}
```

**Component Breakdown:**

| Step | Description |
|------|-------------|
| `this == obj` | Fast path: same reference. |
| `obj == null` | Null check. |
| `getClass() != obj.getClass()` | Exact class check (or `instanceof` for subclass-friendly equality). |
| Cast | Cast to the correct type. |
| Field comparison | Compare significant fields using `==` or `Objects.equals`. |

**Syntax Rules:**

- Always override `hashCode` when overriding `equals`.
- Use `@Override` annotation for compile-time checking.
- Use `Objects.equals` for null-safe field comparison.

**Constraints and Limitations:**

- Violating the contract can cause unpredictable behaviour in collections.
- Using `instanceof` allows equality with subclasses; using `getClass` requires exact class match.

### Annotated Code Example

```java
// File: EqualsDemo.java
import java.util.Objects;

public class EqualsDemo {
    private String name;
    private int age;

    public EqualsDemo(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        EqualsDemo that = (EqualsDemo) o;
        return age == that.age && Objects.equals(name, that.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    public static void main(String[] args) {
        EqualsDemo a = new EqualsDemo("Alice", 30);
        EqualsDemo b = new EqualsDemo("Alice", 30);
        System.out.println(a.equals(b));  // true
    }
}
```

**Expected Output:**
```
true
```

**Step-by-Step Explanation:**

1. `this == o` checks if they are the same reference.
2. `getClass() != o.getClass()` ensures the same type.
3. Fields are compared: `age` with `==`, `name` with `Objects.equals`.
4. `hashCode` is overridden consistently.

### Real-World Cases

- **Entity Classes:** Two `User` objects are equal if they have the same `id`.
- **Value Objects:** `Money`, `Date`, `Point` use value equality.
- **Collections:** `List.contains()`, `Set.add()` rely on `equals`.

---

## 3. hashCode()

### Definitions

**Core Definition:** The `hashCode()` method returns an integer hash code for an object, used in hash-based collections.

**Technical Definition:** "Returns a hash code value for the object. This method is supported for the benefit of hash tables such as those provided by `HashMap`". The general contract requires that if two objects are equal according to `equals`, they must have the same hash code.

**Beginner-Friendly Explanation:** `hashCode()` is like a quick "bucket number" for an object. It helps hash-based collections find objects quickly. If two objects are equal, they must have the same bucket number.

### Purposes

- To enable efficient storage and retrieval in hash-based collections
- To maintain consistency with `equals`
- To provide a fast preliminary check for equality

### Syntax Rules and Structure

```java
@Override
public int hashCode() {
    return Objects.hash(field1, field2, ...);
}
```

**Syntax Rules:**

- If `equals` is overridden, `hashCode` must also be overridden.
- Equal objects must have equal hash codes.
- Unequal objects may share a hash code (collision).

**Constraints and Limitations:**

- Poor hash code distribution degrades collection performance.
- Mutable fields used in `hashCode` can cause problems if modified after insertion into a hash-based collection.

### Annotated Code Example

```java
// File: HashCodeDemo.java
import java.util.Objects;

public class HashCodeDemo {
    private String name;
    private int id;

    public HashCodeDemo(String name, int id) {
        this.name = name;
        this.id = id;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        HashCodeDemo that = (HashCodeDemo) o;
        return id == that.id && Objects.equals(name, that.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, id);
    }

    public static void main(String[] args) {
        HashCodeDemo a = new HashCodeDemo("Alice", 1);
        HashCodeDemo b = new HashCodeDemo("Alice", 1);
        System.out.println("a.hashCode() == b.hashCode(): " + (a.hashCode() == b.hashCode()));
    }
}
```

**Expected Output:**
```
a.hashCode() == b.hashCode(): true
```

**Step-by-Step Explanation:**

1. `equals` compares `name` and `id`.
2. `hashCode` uses `Objects.hash(name, id)`, producing the same hash for equal objects.
3. The output confirms the contract.

### Real-World Cases

- **`HashMap`:** Uses `hashCode` to find buckets, then `equals` to find the exact key.
- **`HashSet`:** Uses `hashCode` to detect duplicates.
- **Caching:** Hash codes speed up lookups.

---

## 4. toString()

### Definitions

**Core Definition:** The `toString()` method returns a string representation of an object.

**Technical Definition:** "Returns a string representation of the object. In general, the `toString` method returns a string that 'textually represents' this object. The result should be a concise but informative representation that is easy for a person to read."

**Beginner-Friendly Explanation:** `toString()` is how an object describes itself as text. When you print an object, Java calls its `toString()` method.

### Purposes

- To provide a human-readable representation of an object
- To aid debugging and logging
- To support string concatenation and formatting

### Syntax Rules and Structure

```java
@Override
public String toString() {
    return "ClassName{field1=" + field1 + ", field2=" + field2 + "}";
}
```

**Syntax Rules:**

- Should return a concise but informative representation.
- Should include significant fields.
- Should not return `null`.

**Constraints and Limitations:**

- The default implementation returns `ClassName@hashCode` in hexadecimal.
- `toString` should not expose sensitive information.

### Annotated Code Example

```java
// File: ToStringDemo.java
public class ToStringDemo {
    private String name;
    private int age;

    public ToStringDemo(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }

    public static void main(String[] args) {
        ToStringDemo p = new ToStringDemo("Alice", 30);
        System.out.println(p);  // Person{name='Alice', age=30}
    }
}
```

**Expected Output:**
```
Person{name='Alice', age=30}
```

**Step-by-Step Explanation:**

1. The `toString` method returns a formatted string with field values.
2. `System.out.println(p)` calls `p.toString()` implicitly.
3. The output shows a readable representation.

### Real-World Cases

- **Logging:** `logger.info("User: " + user)` calls `toString`.
- **Debugging:** IDEs display `toString` in debuggers.
- **Error Messages:** Exceptions include object descriptions.

---

## 5. getClass()

### Definitions

**Core Definition:** The `getClass()` method returns the runtime class of an object.

**Technical Definition:** "Returns the runtime class of this `Object`. The returned `Class` object is the object that is locked by `static synchronized` methods of the represented class". The method is declared `public final Class<?> getClass()`.

**Beginner-Friendly Explanation:** `getClass()` tells you what type an object really is at runtime, which may be different from the type of the reference variable pointing to it.

### Purposes

- To determine the exact runtime type of an object
- To enable reflection-based programming
- To support type checks and debugging

### Syntax Rules and Structure

```java
public final Class<?> getClass()
```

**Syntax Rules:**

- `getClass()` is `final` and cannot be overridden.
- Returns a `Class<?>` object representing the runtime class.
- The actual result type is `Class<? extends |X|>` where `|X|` is the static type of the expression.

**Constraints and Limitations:**

- Cannot be overridden.
- The returned `Class` object is the same for all instances of a class.

### Annotated Code Example

```java
// File: GetClassDemo.java
public class GetClassDemo {
    public static void main(String[] args) {
        Object obj = "Hello";
        System.out.println("Runtime class: " + obj.getClass().getName());  // java.lang.String

        Object num = 42;
        System.out.println("Runtime class: " + num.getClass().getName());  // java.lang.Integer
    }
}
```

**Expected Output:**
```
Runtime class: java.lang.String
Runtime class: java.lang.Integer
```

**Step-by-Step Explanation:**

1. `obj` is declared as `Object` but holds a `String`.
2. `obj.getClass().getName()` returns `"java.lang.String"` because that is the actual runtime type.
3. Similarly, `num` holds an `Integer`, so `getClass()` returns `"java.lang.Integer"`.

### Real-World Cases

- **Reflection:** `Class.forName()` and `getClass()` are entry points to reflection.
- **Logging Frameworks:** Determining the actual class for log messages.
- **Serialization:** Identifying the class during deserialization.

---

## 6. clone()

### Definitions

**Core Definition:** The `clone()` method creates and returns a copy of an object.

**Technical Definition:** "Creates and returns a copy of this object. The precise meaning of 'copy' may depend on the class of the object". "The method `clone` for class `Object` performs a specific cloning operation. First, if the class of this object does not implement the interface `Cloneable`, then a `CloneNotSupportedException` is thrown. ... Otherwise, this method creates a new instance of the class of this object and initializes all its fields with exactly the contents of the corresponding fields of this object ... this method performs a 'shallow copy' of this object, not a 'deep copy' operation".

**Beginner-Friendly Explanation:** `clone()` is like making a photocopy of an object. The copy has the same data, but it is a separate object. However, if the object contains references to other objects, the copy will point to the same nested objects (shallow copy), not copies of them.

### Purposes

- To create copies of objects without invoking constructors
- To implement the Prototype pattern
- To support defensive copying

### Syntax Rules and Structure

```java
@Override
public Object clone() throws CloneNotSupportedException {
    return super.clone();
}
```

**Syntax Rules:**

- Must implement `Cloneable` or throw `CloneNotSupportedException`.
- `super.clone()` performs a shallow copy.
- Mutable fields must be cloned manually for a deep copy.

**Constraints and Limitations:**

- `clone()` is considered problematic; many experts recommend copy constructors instead.
- Arrays are considered to implement `Cloneable`.

### Annotated Code Example

```java
// File: CloneDemo.java
public class CloneDemo implements Cloneable {
    private int[] data;

    public CloneDemo(int[] data) {
        this.data = data;
    }

    @Override
    public CloneDemo clone() throws CloneNotSupportedException {
        CloneDemo copy = (CloneDemo) super.clone();
        copy.data = data.clone();  // Deep copy of the array
        return copy;
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        CloneDemo original = new CloneDemo(new int[]{1, 2, 3});
        CloneDemo copy = original.clone();
        copy.data[0] = 99;
        System.out.println("Original[0]: " + original.data[0]);  // 1
        System.out.println("Copy[0]: " + copy.data[0]);          // 99
    }
}
```

**Expected Output:**
```
Original[0]: 1
Copy[0]: 99
```

**Step-by-Step Explanation:**

1. `super.clone()` creates a shallow copy.
2. `copy.data = data.clone()` creates a deep copy of the array.
3. Modifying the copy's array does not affect the original.

### Real-World Cases

- **Prototype Pattern:** Creating new objects by cloning prototypes.
- **Defensive Copying:** Returning copies of internal arrays.
- **Avoiding Constructors:** When constructors are expensive.

---

## 7. finalize() (deprecated)

### Definitions

**Core Definition:** The `finalize()` method was called by the garbage collector before an object was reclaimed, allowing cleanup. It is now deprecated for removal.

**Technical Definition:** "Called by the garbage collector on an object when garbage collection determines that there are no more references to the object. A subclass overrides the `finalize` method to dispose of system resources or to perform other cleanup". As of JDK 18 (JEP 421), "Finalization is deprecated and subject to removal in a future release. The use of finalization can lead to problems with security, performance, and reliability".

**Beginner-Friendly Explanation:** `finalize()` was Java's old way of cleaning up resources (like closing files) before an object was destroyed. But it was unreliable and dangerous, so it has been deprecated. Modern Java uses `try-with-resources` and the `Cleaner` API instead.

### Purposes

- **Deprecated.** Formerly used to release resources before an object was garbage collected
- Now serves as a warning: use alternatives like `AutoCloseable` and `Cleaner`

### Syntax Rules and Structure

```java
@Deprecated(since="9", forRemoval=true)
protected void finalize() throws Throwable { ... }
```

**Syntax Rules:**

- Do not override `finalize()` in new code.
- Use `Cleaner` and `PhantomReference` for resource cleanup.
- Implement `AutoCloseable` and use try-with-resources for deterministic cleanup.

**Constraints and Limitations:**

- Finalization can lead to security, performance, and reliability problems.
- JDK 27 will remove the empty method.

### Annotated Code Example (Demonstrating Deprecation)

```java
// File: FinalizeDemo.java
public class FinalizeDemo {
    @Override
    @SuppressWarnings("deprecation")
    protected void finalize() throws Throwable {
        try {
            System.out.println("Finalize called");
        } finally {
            super.finalize();
        }
    }

    public static void main(String[] args) {
        FinalizeDemo obj = new FinalizeDemo();
        obj = null;
        System.gc();
        System.out.println("End of main");
    }
}
```

**Expected Output:**
```
End of main
Finalize called
```

**Step-by-Step Explanation:**

1. The `finalize()` method is overridden (but this is strongly discouraged).
2. When `obj` is set to `null` and `System.gc()` is called, the garbage collector may eventually call `finalize()`.
3. The output order is unpredictable because garbage collection timing is non-deterministic.

### Real-World Cases

- **Legacy Code:** Old codebases may still use `finalize()`, but it should be replaced.
- **Migration:** Use `Cleaner` (Java 9+) or `try-with-resources` for resource management.

---

## 8. wait()

### Definitions

**Core Definition:** The `wait()` method causes the current thread to wait until another thread invokes `notify()` or `notifyAll()` on the same object.

**Technical Definition:** "Causes the current thread to wait until another thread invokes the `notify()` method or the `notifyAll()` method for this object, or some other thread interrupts the current thread, or a certain amount of real time has elapsed".

**Beginner-Friendly Explanation:** `wait()` tells a thread to pause and wait for a signal from another thread. It's like waiting in a waiting room until someone calls your name.

### Purposes

- To coordinate thread execution in producer-consumer scenarios
- To allow threads to wait for a condition to become true
- To implement inter-thread communication

### Syntax Rules and Structure

```java
public final void wait() throws InterruptedException
public final void wait(long timeout) throws InterruptedException
public final void wait(long timeout, int nanos) throws InterruptedException
```

**Syntax Rules:**

- Must be called from within a `synchronized` block or method.
- The thread must own the object's monitor.
- Throws `InterruptedException` if interrupted while waiting.

**Constraints and Limitations:**

- `wait()` is `final` and cannot be overridden.
- Spurious wakeups can occur; always use `wait()` in a loop with a condition check.

### Annotated Code Example

```java
// File: WaitDemo.java
public class WaitDemo {
    private static final Object lock = new Object();
    private static boolean ready = false;

    public static void main(String[] args) throws InterruptedException {
        Thread waiter = new Thread(() -> {
            synchronized (lock) {
                while (!ready) {
                    try {
                        System.out.println("Waiting...");
                        lock.wait();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
                System.out.println("Ready!");
            }
        });

        Thread notifier = new Thread(() -> {
            synchronized (lock) {
                ready = true;
                lock.notify();
                System.out.println("Notified!");
            }
        });

        waiter.start();
        Thread.sleep(100);  // Ensure waiter is waiting
        notifier.start();
    }
}
```

**Expected Output:**
```
Waiting...
Notified!
Ready!
```

**Step-by-Step Explanation:**

1. The `waiter` thread enters a synchronized block and calls `wait()` because `ready` is `false`.
2. The `notifier` thread enters a synchronized block, sets `ready = true`, and calls `notify()`.
3. The `waiter` thread wakes up, checks `ready` again (loop), and prints "Ready!".

### Real-World Cases

- **Producer-Consumer:** Consumers wait for items; producers notify when items are added.
- **Thread Pools:** Worker threads wait for tasks.
- **Event Handling:** Threads wait for events to occur.

---

## 9. notify()

### Definitions

**Core Definition:** The `notify()` method wakes up a single thread that is waiting on the object's monitor.

**Technical Definition:** "Wakes up a single thread that is waiting on this object's monitor. If any threads are waiting on this object, one of them is chosen to be awakened. The choice is arbitrary and occurs at the discretion of the implementation".

**Beginner-Friendly Explanation:** `notify()` wakes up one thread that is waiting. It's like calling one person's name in a waiting room.

### Purposes

- To wake up a single waiting thread
- To signal that a condition has changed
- To implement inter-thread communication

### Syntax Rules and Structure

```java
public final void notify()
```

**Syntax Rules:**

- Must be called from within a `synchronized` block or method.
- The thread must own the object's monitor.
- Wakes up one arbitrary waiting thread.

**Constraints and Limitations:**

- `notify()` is `final` and cannot be overridden.
- The awakened thread cannot proceed until the current thread relinquishes the lock.

### Annotated Code Example

See `WaitDemo` above for a combined example of `wait()` and `notify()`.

### Real-World Cases

- **Producer-Consumer:** Notifying a single consumer that an item is available.
- **Thread Coordination:** Signaling one worker thread to proceed.

---

## 10. notifyAll()

### Definitions

**Core Definition:** The `notifyAll()` method wakes up all threads that are waiting on the object's monitor.

**Technical Definition:** "Wakes up all threads that are waiting on this object's monitor. A thread waits on an object's monitor by calling one of the `wait` methods. The awakened threads will not be able to proceed until the current thread relinquishes the lock on this object".

**Beginner-Friendly Explanation:** `notifyAll()` wakes up every thread that is waiting. It's like announcing "Everyone, wake up!" in a waiting room.

### Purposes

- To wake up all waiting threads when a condition changes that affects all of them
- To prevent starvation when multiple threads are waiting
- To implement broadcast-style communication

### Syntax Rules and Structure

```java
public final void notifyAll()
```

**Syntax Rules:**

- Must be called from within a `synchronized` block or method.
- The thread must own the object's monitor.
- Wakes up all waiting threads.

**Constraints and Limitations:**

- `notifyAll()` is `final` and cannot be overridden.
- More costly than `notify()` when many threads are waiting.

### Annotated Code Example

```java
// File: NotifyAllDemo.java
public class NotifyAllDemo {
    private static final Object lock = new Object();
    private static boolean ready = false;

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            final int id = i;
            new Thread(() -> {
                synchronized (lock) {
                    while (!ready) {
                        try {
                            System.out.println("Thread " + id + " waiting");
                            lock.wait();
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                        }
                    }
                    System.out.println("Thread " + id + " ready");
                }
            }).start();
        }

        Thread.sleep(100);
        synchronized (lock) {
            ready = true;
            lock.notifyAll();
        }
    }
}
```

**Expected Output:**
```
Thread 0 waiting
Thread 1 waiting
Thread 2 waiting
Thread 0 ready
Thread 1 ready
Thread 2 ready
```

**Step-by-Step Explanation:**

1. Three threads wait on the lock.
2. The main thread sets `ready = true` and calls `notifyAll()`.
3. All three threads wake up, check the condition, and print "ready".

### Real-World Cases

- **Thread Pools:** Notifying all worker threads that tasks are available.
- **Broadcast Events:** Notifying all listeners that an event occurred.
- **Resource Pools:** Notifying all threads that a resource is available.

---

## 11. Overriding equals()

### Definitions

**Core Definition:** Overriding `equals()` means providing a custom implementation for logical equality in a class.

**Technical Definition:** When overriding `equals`, the method must be reflexive, symmetric, transitive, consistent, and return `false` for `null`. It should also be consistent with `hashCode()`.

**Beginner-Friendly Explanation:** Overriding `equals()` lets you define what it means for two objects of your class to be "equal" based on their contents, not just their memory addresses.

### Purposes

- To define domain-specific equality rules
- To enable correct behaviour in collections
- To support value-based comparison

### Syntax Rules and Structure

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    ClassName other = (ClassName) obj;
    return field1 == other.field1 && Objects.equals(field2, other.field2);
}
```

**Syntax Rules:**

- Use `@Override` annotation.
- Always override `hashCode` when overriding `equals`.
- Use `Objects.equals` for null-safe field comparison.

**Constraints and Limitations:**

- Violating the contract can cause unpredictable behaviour in collections.
- Using `instanceof` allows equality with subclasses; using `getClass` requires exact class match.

### Annotated Code Example

See `EqualsDemo` in Section 2.

### Real-World Cases

- **Entity Classes:** Equality based on primary key.
- **Value Objects:** Equality based on all significant fields.

---

## 12. Overriding hashCode()

### Definitions

**Core Definition:** Overriding `hashCode()` means providing a custom hash code computation for a class.

**Technical Definition:** The `hashCode` method must return the same value for two objects that are equal according to `equals()`. It should also produce different values for unequal objects as much as possible (to minimise collisions).

**Beginner-Friendly Explanation:** Overriding `hashCode()` lets you control how your objects are bucketed in hash-based collections, improving performance and ensuring correctness.

### Purposes

- To ensure consistency with `equals()`
- To enable efficient retrieval in hash-based collections
- To minimise hash collisions

### Syntax Rules and Structure

```java
@Override
public int hashCode() {
    return Objects.hash(field1, field2, ...);
}
```

**Syntax Rules:**

- Use `Objects.hash()` for convenience.
- Include the same fields used in `equals()`.
- For performance-critical code, consider manual computation.

**Constraints and Limitations:**

- Poor hash code distribution degrades collection performance.
- Mutable fields used in `hashCode` can cause problems if modified after insertion into a hash-based collection.

### Annotated Code Example

See `HashCodeDemo` in Section 3.

### Real-World Cases

- **`HashMap` keys:** Correct `hashCode` ensures fast lookups.
- **`HashSet`:** Correct `hashCode` ensures duplicates are detected.

---

## 13. equals() and hashCode() contract

### Definitions

**Core Definition:** The `equals()` and `hashCode()` contract is a set of rules that ensure these two methods behave consistently.

**Technical Definition:** The contract states: (1) If two objects are equal according to `equals(Object)`, then calling `hashCode` on each must produce the same integer result. (2) `equals` must define an equivalence relation (reflexive, symmetric, transitive). (3) `equals` must be consistent. (4) `equals` must return `false` for `null`. Failure to follow this contract is a common source of defects.

**Beginner-Friendly Explanation:** The contract says: "If two objects are equal, they must have the same hash code." If you break this rule, hash-based collections will behave unpredictably — you might put an object in a `HashMap` and not be able to find it again.

### Purposes

- To ensure correct behaviour in hash-based collections
- To prevent subtle and hard-to-debug errors
- To maintain consistency between equality and hashing

### Syntax Rules and Structure

**The Contract Rules:**

| Rule | Description |
|------|-------------|
| Reflexive | `x.equals(x)` must return `true`. |
| Symmetric | `x.equals(y)` must return `true` iff `y.equals(x)` returns `true`. |
| Transitive | If `x.equals(y)` and `y.equals(z)`, then `x.equals(z)`. |
| Consistent | `x.equals(y)` must consistently return the same result. |
| Null | `x.equals(null)` must return `false`. |
| hashCode consistency | Equal objects must have equal hash codes. |

**Syntax Rules:**

- Always override both `equals` and `hashCode` together.
- Use the same fields in both methods.
- Test equality and hash code consistency thoroughly.

**Constraints and Limitations:**

- Violating the contract can cause data loss in collections.
- Immutable objects are exempt from some concerns.

### Annotated Code Example

See the SEI CERT example in the search results, which demonstrates the defect of overriding `equals` without `hashCode`.

### Real-World Cases

- **`HashMap`:** Objects that override `equals` but not `hashCode` will not be found.
- **`HashSet`:** Duplicates may be allowed if `hashCode` is inconsistent.

---

## 14. Objects utility class

### Definitions

**Core Definition:** The `Objects` class is a utility class (introduced in Java 7) that provides static methods for operating on objects, with a focus on null safety.

**Technical Definition:** "This class consists of `static` utility methods for operating on objects, or checking certain conditions before operation. These utilities include `null`-safe or `null`-tolerant methods for computing the hash code of an object, returning a string for an object, comparing two objects, and checking if indexes or sub-range values are out of bounds".

**Beginner-Friendly Explanation:** `Objects` is a toolbox of helper methods that make working with objects easier, especially when they might be `null`. It saves you from writing repetitive null-checking code.

### Purposes

- To provide null-safe operations on objects
- To simplify common tasks like hashing, string conversion, and comparison
- To reduce boilerplate code and `NullPointerException` risks
- To offer validation utilities for method parameters

### Syntax Rules and Structure

```java
// All methods are static
Objects.methodName(args);
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `equals(Object a, Object b)` | Null-safe equality check. |
| `hash(Object... values)` | Generates a hash code for a sequence of values. |
| `hashCode(Object o)` | Returns the hash code of a non-null argument. |
| `toString(Object o)` | Returns the string representation of an object. |
| `requireNonNull(T obj)` | Throws NPE if obj is null. |
| `requireNonNullElse(T obj, T defaultObj)` | Returns obj if non-null, else defaultObj. |
| `isNull(Object obj)` | Returns true if obj is null. |
| `nonNull(Object obj)` | Returns true if obj is non-null. |
| `checkIndex(int index, int length)` | Checks if index is within bounds. |
| `checkFromToIndex(int fromIndex, int toIndex, int length)` | Checks sub-range bounds. |
| `compare(T a, T b, Comparator c)` | Compares two objects using a comparator. |

**Syntax Rules:**

- All methods are `static`.
- The class is `final` and cannot be instantiated.
- Methods are designed for null safety.

**Constraints and Limitations:**

- Some methods (like `requireNonNullElse`) throw `NullPointerException` if both arguments are null.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Null-Safe Operations**

```java
// File: ObjectsUtilityDemo.java
import java.util.Objects;

public class ObjectsUtilityDemo {
    public static void main(String[] args) {
        String a = null;
        String b = "Hello";

        // equals - null-safe
        System.out.println(Objects.equals(a, b));  // false
        System.out.println(Objects.equals(null, null));  // true

        // hashCode - null-safe
        System.out.println(Objects.hashCode(a));  // 0
        System.out.println(Objects.hashCode(b));  // hash of "Hello"

        // toString - null-safe
        System.out.println(Objects.toString(a));  // "null"
        System.out.println(Objects.toString(a, "default"));  // "default"

        // requireNonNull
        try {
            Objects.requireNonNull(a, "a must not be null");
        } catch (NullPointerException e) {
            System.out.println("Caught: " + e.getMessage());  // Caught: a must not be null
        }

        // requireNonNullElse
        System.out.println(Objects.requireNonNullElse(a, "fallback"));  // fallback

        // isNull / nonNull
        System.out.println(Objects.isNull(a));   // true
        System.out.println(Objects.nonNull(b));  // true
    }
}
```

**Expected Output:**
```
false
true
0
69609650
null
default
Caught: a must not be null
fallback
true
true
```

**Step-by-Step Explanation:**

1. `Objects.equals(a, b)` safely compares `null` to `"Hello"`, returning `false`.
2. `Objects.hashCode(a)` returns `0` for `null`.
3. `Objects.toString(a, "default")` returns `"default"` because `a` is `null`.
4. `Objects.requireNonNull(a, "...")` throws `NullPointerException` with a custom message.
5. `Objects.requireNonNullElse(a, "fallback")` returns `"fallback"`.
6. `Objects.isNull(a)` returns `true`; `Objects.nonNull(b)` returns `true`.

### Real-World Cases

- **Method Parameter Validation:** `Objects.requireNonNull(param, "param")` at the start of methods.
- **Hash Code Computation:** `Objects.hash(field1, field2)` in `hashCode()` overrides.
- **Null-Safe Comparison:** `Objects.equals(a, b)` in `equals()` overrides.
- **Index Validation:** `Objects.checkIndex(index, length)` in collection implementations.

---

## 15. Objects.equals()

### Definitions

**Core Definition:** `Objects.equals(Object a, Object b)` returns `true` if the arguments are equal to each other and `false` otherwise.

**Technical Definition:** "Returns `true` if the arguments are equal to each other and `false` otherwise. Consequently, if both arguments are `null`, `true` is returned and if exactly one argument is `null`, `false` is returned."

**Beginner-Friendly Explanation:** `Objects.equals()` is a null-safe way to compare two objects. It handles `null` gracefully, unlike calling `a.equals(b)` directly.

### Purposes

- To perform null-safe equality comparison
- To avoid `NullPointerException` when comparing objects
- To simplify `equals()` implementations

### Syntax Rules and Structure

```java
public static boolean equals(Object a, Object b)
```

**Syntax Rules:**

- Returns `true` if both are `null`.
- Returns `false` if exactly one is `null`.
- Otherwise, returns `a.equals(b)`.

**Constraints and Limitations:**

- None significant; it is a convenience method.

### Annotated Code Example

```java
// File: ObjectsEqualsDemo.java
import java.util.Objects;

public class ObjectsEqualsDemo {
    public static void main(String[] args) {
        System.out.println(Objects.equals(null, null));        // true
        System.out.println(Objects.equals(null, "Hello"));     // false
        System.out.println(Objects.equals("Hello", "Hello"));  // true
    }
}
```

**Expected Output:**
```
true
false
true
```

**Step-by-Step Explanation:**

1. Both `null` → `true`.
2. One `null` → `false`.
3. Both equal strings → `true`.

### Real-World Cases

- **`equals()` Implementation:** `return Objects.equals(this.name, other.name);`
- **Null-Safe Comparison:** Anywhere two objects need to be compared safely.

---

## 16. Objects.hash()

### Definitions

**Core Definition:** `Objects.hash(Object... values)` generates a hash code for a sequence of input values.

**Technical Definition:** "Generates a hash code for a sequence of input values. The hash code is generated as if all the input values were placed into an array, and that array were hashed by calling `Arrays.hashCode(Object[])`."

**Beginner-Friendly Explanation:** `Objects.hash()` is a convenient way to compute a hash code from multiple fields without writing manual arithmetic.

### Purposes

- To simplify `hashCode()` implementations
- To generate consistent hash codes from multiple fields
- To reduce boilerplate code

### Syntax Rules and Structure

```java
public static int hash(Object... values)
```

**Syntax Rules:**

- Accepts a variable number of arguments.
- Returns an integer hash code.
- Handles `null` values safely.

**Constraints and Limitations:**

- Slightly slower than manual computation due to varargs array creation.
- Not suitable for performance-critical code with many fields.

### Annotated Code Example

```java
// File: ObjectsHashDemo.java
import java.util.Objects;

public class ObjectsHashDemo {
    private String name;
    private int age;

    public ObjectsHashDemo(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    public static void main(String[] args) {
        ObjectsHashDemo a = new ObjectsHashDemo("Alice", 30);
        ObjectsHashDemo b = new ObjectsHashDemo("Alice", 30);
        System.out.println(a.hashCode() == b.hashCode());  // true
    }
}
```

**Expected Output:**
```
true
```

**Step-by-Step Explanation:**

1. `Objects.hash(name, age)` computes a hash from both fields.
2. Equal objects produce equal hash codes.

### Real-World Cases

- **`hashCode()` Overrides:** The standard way to implement `hashCode()` in modern Java.
- **Data Classes:** Automatically generated in records and Lombok `@Data` classes.

---

## 17. Objects.hashCode()

### Definitions

**Core Definition:** `Objects.hashCode(Object o)` returns the hash code of a non-null argument and `0` for a null argument.

**Technical Definition:** "Returns the hash code of a non-`null` argument and `0` for a `null` argument."

**Beginner-Friendly Explanation:** `Objects.hashCode()` is a null-safe version of `o.hashCode()`. It returns `0` if the object is `null`.

### Purposes

- To safely obtain a hash code without null checks
- To avoid `NullPointerException`
- To simplify hash code computation for single fields

### Syntax Rules and Structure

```java
public static int hashCode(Object o)
```

**Syntax Rules:**

- Returns `0` if `o` is `null`.
- Otherwise, returns `o.hashCode()`.

**Constraints and Limitations:**

- Only for single objects; use `Objects.hash()` for multiple fields.

### Annotated Code Example

```java
// File: ObjectsHashCodeDemo.java
import java.util.Objects;

public class ObjectsHashCodeDemo {
    public static void main(String[] args) {
        String s = null;
        System.out.println(Objects.hashCode(s));  // 0
        System.out.println(Objects.hashCode("Hello"));  // hash of "Hello"
    }
}
```

**Expected Output:**
```
0
69609650
```

**Step-by-Step Explanation:**

1. `Objects.hashCode(null)` returns `0`.
2. `Objects.hashCode("Hello")` returns the hash code of the string.

### Real-World Cases

- **Single-Field `hashCode`:** When a class has only one significant field.

---

## 18. Objects.toString()

### Definitions

**Core Definition:** `Objects.toString(Object o)` returns the string representation of an object, with overloads that provide a default value.

**Technical Definition:** "Returns the result of calling `toString` on the first argument if the first argument is not `null` and returns the second argument otherwise."

**Beginner-Friendly Explanation:** `Objects.toString()` is a null-safe way to get an object's string representation. You can provide a fallback string for `null` objects.

### Purposes

- To safely obtain a string representation without null checks
- To provide a default string for null objects
- To simplify logging and debugging

### Syntax Rules and Structure

```java
public static String toString(Object o)
public static String toString(Object o, String nullDefault)
```

**Syntax Rules:**

- `toString(o)` returns `"null"` if `o` is `null`.
- `toString(o, nullDefault)` returns `nullDefault` if `o` is `null`.

**Constraints and Limitations:**

- None significant.

### Annotated Code Example

```java
// File: ObjectsToStringDemo.java
import java.util.Objects;

public class ObjectsToStringDemo {
    public static void main(String[] args) {
        String s = null;
        System.out.println(Objects.toString(s));             // "null"
        System.out.println(Objects.toString(s, "unknown"));  // "unknown"
        System.out.println(Objects.toString("Hello"));       // "Hello"
    }
}
```

**Expected Output:**
```
null
unknown
Hello
```

**Step-by-Step Explanation:**

1. `Objects.toString(null)` returns `"null"`.
2. `Objects.toString(null, "unknown")` returns `"unknown"`.
3. `Objects.toString("Hello")` returns `"Hello"`.

### Real-World Cases

- **Logging:** Safely logging objects that might be `null`.
- **String Building:** Safe string concatenation.

---

## 19. Objects.requireNonNull()

### Definitions

**Core Definition:** `Objects.requireNonNull(T obj)` checks that the specified object reference is not `null` and throws a `NullPointerException` if it is.

**Technical Definition:** "Checks that the specified object reference is not `null`. This method is designed primarily for doing parameter validation in methods and constructors."

**Beginner-Friendly Explanation:** `Objects.requireNonNull()` is a guard method that ensures an object is not `null`. If it is `null`, it throws an exception immediately.

### Purposes

- To validate method and constructor parameters
- To fail fast when `null` is passed where it is not allowed
- To provide clear error messages

### Syntax Rules and Structure

```java
public static <T> T requireNonNull(T obj)
public static <T> T requireNonNull(T obj, String message)
```

**Syntax Rules:**

- Returns `obj` if non-null.
- Throws `NullPointerException` if `obj` is `null`.
- The optional message is included in the exception.

**Constraints and Limitations:**

- Throws `NullPointerException` (a runtime exception).

### Annotated Code Example

```java
// File: RequireNonNullDemo.java
import java.util.Objects;

public class RequireNonNullDemo {
    private String name;

    public RequireNonNullDemo(String name) {
        this.name = Objects.requireNonNull(name, "Name cannot be null");
    }

    public static void main(String[] args) {
        try {
            new RequireNonNullDemo(null);
        } catch (NullPointerException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Caught: Name cannot be null
```

**Step-by-Step Explanation:**

1. `Objects.requireNonNull(name, "...")` checks if `name` is `null`.
2. If `null`, it throws `NullPointerException` with the custom message.
3. The exception is caught and the message is printed.

### Real-World Cases

- **Constructor Validation:** Ensuring required dependencies are provided.
- **Method Parameter Validation:** Checking inputs at the start of methods.
- **Fail-Fast Design:** Detecting `null` early rather than later.

---

## 20. Objects.requireNonNullElse()

### Definitions

**Core Definition:** `Objects.requireNonNullElse(T obj, T defaultObj)` returns the first argument if it is non-null and otherwise returns the non-null second argument.

**Technical Definition:** "Returns the first argument if it is non-`null` and otherwise returns the non-`null` second argument." If both arguments are `null`, a `NullPointerException` is thrown.

**Beginner-Friendly Explanation:** `Objects.requireNonNullElse()` is like a fallback: "Use this value if it's not null; otherwise use this default."

### Purposes

- To provide a default value when an object is `null`
- To simplify null-coalescing logic
- To ensure a non-null result

### Syntax Rules and Structure

```java
public static <T> T requireNonNullElse(T obj, T defaultObj)
```

**Syntax Rules:**

- Returns `obj` if non-null.
- Returns `defaultObj` if `obj` is null and `defaultObj` is non-null.
- Throws `NullPointerException` if both are null.

**Constraints and Limitations:**

- Both arguments cannot be `null`.

### Annotated Code Example

```java
// File: RequireNonNullElseDemo.java
import java.util.Objects;

public class RequireNonNullElseDemo {
    public static void main(String[] args) {
        String a = null;
        String b = "Default";

        System.out.println(Objects.requireNonNullElse(a, b));  // Default
        System.out.println(Objects.requireNonNullElse("Value", b));  // Value

        try {
            Objects.requireNonNullElse(null, null);
        } catch (NullPointerException e) {
            System.out.println("Both null: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Default
Value
Both null: null
```

**Step-by-Step Explanation:**

1. `requireNonNullElse(null, "Default")` returns `"Default"`.
2. `requireNonNullElse("Value", "Default")` returns `"Value"`.
3. `requireNonNullElse(null, null)` throws `NullPointerException`.

### Real-World Cases

- **Default Configuration:** Providing fallback values for optional settings.
- **API Design:** Ensuring methods always return non-null values.

---

## 21. Objects.isNull()

### Definitions

**Core Definition:** `Objects.isNull(Object obj)` returns `true` if the provided reference is `null`, otherwise `false`.

**Technical Definition:** "Returns `true` if the provided reference is `null` otherwise returns `false`."

**Beginner-Friendly Explanation:** `Objects.isNull()` is a null-safe way to check if an object is `null`. It's often used in lambda expressions and method references.

### Purposes

- To check for null in a functional style
- To use in stream operations (`filter(Objects::isNull)`)
- To improve readability in some contexts

### Syntax Rules and Structure

```java
public static boolean isNull(Object obj)
```

**Syntax Rules:**

- Returns `true` if `obj` is `null`.
- Returns `false` otherwise.

**Constraints and Limitations:**

- For direct null checks, `obj == null` is simpler and clearer.

### Annotated Code Example

```java
// File: IsNullDemo.java
import java.util.Objects;

public class IsNullDemo {
    public static void main(String[] args) {
        String s = null;
        System.out.println(Objects.isNull(s));      // true
        System.out.println(Objects.isNull("Hello")); // false
    }
}
```

**Expected Output:**
```
true
false
```

**Step-by-Step Explanation:**

1. `Objects.isNull(null)` returns `true`.
2. `Objects.isNull("Hello")` returns `false`.

### Real-World Cases

- **Stream Filtering:** `stream.filter(Objects::isNull)` to find null elements.
- **Functional Programming:** Using method references.

---

## 22. Objects.nonNull()

### Definitions

**Core Definition:** `Objects.nonNull(Object obj)` returns `true` if the provided reference is non-null, otherwise `false`.

**Technical Definition:** "Returns `true` if the provided reference is non-`null` otherwise returns `false`."

**Beginner-Friendly Explanation:** `Objects.nonNull()` is the opposite of `Objects.isNull()`. It checks if an object is not `null`.

### Purposes

- To check for non-null in a functional style
- To use in stream operations (`filter(Objects::nonNull)`)
- To improve readability

### Syntax Rules and Structure

```java
public static boolean nonNull(Object obj)
```

**Syntax Rules:**

- Returns `true` if `obj` is non-null.
- Returns `false` otherwise.

**Constraints and Limitations:**

- For direct null checks, `obj != null` is simpler and clearer.

### Annotated Code Example

```java
// File: NonNullDemo.java
import java.util.Objects;

public class NonNullDemo {
    public static void main(String[] args) {
        String s = null;
        System.out.println(Objects.nonNull(s));      // false
        System.out.println(Objects.nonNull("Hello")); // true
    }
}
```

**Expected Output:**
```
false
true
```

**Step-by-Step Explanation:**

1. `Objects.nonNull(null)` returns `false`.
2. `Objects.nonNull("Hello")` returns `true`.

### Real-World Cases

- **Stream Filtering:** `stream.filter(Objects::nonNull)` to remove null elements.
- **Functional Programming:** Using method references.

---

## 23. Objects.checkIndex()

### Definitions

**Core Definition:** `Objects.checkIndex(int index, int length)` checks if the index is within the bounds of the range from `0` (inclusive) to `length` (exclusive).

**Technical Definition:** "Checks if the `index` is within the bounds of the range from `0` (inclusive) to `length` (exclusive)." Throws `IndexOutOfBoundsException` if the check fails.

**Beginner-Friendly Explanation:** `Objects.checkIndex()` validates that an index is valid for an array or list of a given length. It throws an exception if the index is out of bounds.

### Purposes

- To validate index bounds before accessing an array or list
- To provide a consistent exception type for index errors
- To simplify code that manually checks index bounds

### Syntax Rules and Structure

```java
public static int checkIndex(int index, int length)
public static long checkIndex(long index, long length)
```

**Syntax Rules:**

- Returns `index` if valid.
- Throws `IndexOutOfBoundsException` if `index < 0` or `index >= length`.

**Constraints and Limitations:**

- Throws `IndexOutOfBoundsException`, not `ArrayIndexOutOfBoundsException` or `StringIndexOutOfBoundsException`.

### Annotated Code Example

```java
// File: CheckIndexDemo.java
import java.util.Objects;

public class CheckIndexDemo {
    public static void main(String[] args) {
        int[] arr = {10, 20, 30};

        try {
            int value = arr[Objects.checkIndex(1, arr.length)];
            System.out.println("Value: " + value);  // Value: 20
        } catch (IndexOutOfBoundsException e) {
            System.out.println("Invalid index");
        }

        try {
            arr[Objects.checkIndex(5, arr.length)];
        } catch (IndexOutOfBoundsException e) {
            System.out.println("Caught: index out of bounds");  // Caught: index out of bounds
        }
    }
}
```

**Expected Output:**
```
Value: 20
Caught: index out of bounds
```

**Step-by-Step Explanation:**

1. `Objects.checkIndex(1, 3)` returns `1` because it is valid.
2. `Objects.checkIndex(5, 3)` throws `IndexOutOfBoundsException`.

### Real-World Cases

- **Collection Implementations:** Validating indices in custom `List` implementations.
- **Array Utilities:** Safe array access.

---

## 24. Objects.checkFromToIndex()

### Definitions

**Core Definition:** `Objects.checkFromToIndex(int fromIndex, int toIndex, int length)` checks if the sub-range from `fromIndex` (inclusive) to `toIndex` (exclusive) is within the bounds of range from `0` (inclusive) to `length` (exclusive).

**Technical Definition:** "Checks if the sub-range from `fromIndex` (inclusive) to `toIndex` (exclusive) is within the bounds of range from `0` (inclusive) to `length` (exclusive)." Throws `IndexOutOfBoundsException` if the check fails.

**Beginner-Friendly Explanation:** `Objects.checkFromToIndex()` validates a sub-range (like a slice) of an array or list.

### Purposes

- To validate sub-range bounds for operations like `Arrays.copyOfRange`
- To provide consistent exception handling
- To simplify sub-range validation

### Syntax Rules and Structure

```java
public static int checkFromToIndex(int fromIndex, int toIndex, int length)
public static long checkFromToIndex(long fromIndex, long toIndex, long length)
```

**Syntax Rules:**

- Throws `IndexOutOfBoundsException` if `fromIndex < 0`, `toIndex > length`, or `fromIndex > toIndex`.

### Annotated Code Example

```java
// File: CheckFromToIndexDemo.java
import java.util.Objects;

public class CheckFromToIndexDemo {
    public static void main(String[] args) {
        int length = 10;

        try {
            Objects.checkFromToIndex(2, 5, length);
            System.out.println("Valid range");
        } catch (IndexOutOfBoundsException e) {
            System.out.println("Invalid range");
        }

        try {
            Objects.checkFromToIndex(5, 2, length);
        } catch (IndexOutOfBoundsException e) {
            System.out.println("Caught: from > to");  // Caught: from > to
        }
    }
}
```

**Expected Output:**
```
Valid range
Caught: from > to
```

**Step-by-Step Explanation:**

1. `checkFromToIndex(2, 5, 10)` passes because `2 <= 5 <= 10`.
2. `checkFromToIndex(5, 2, 10)` fails because `fromIndex > toIndex`.

### Real-World Cases

- **Array Slicing:** Validating ranges for `Arrays.copyOfRange`.
- **Sub-list Operations:** Validating ranges for `List.subList`.

---

## 25. Objects.compare()

### Definitions

**Core Definition:** `Objects.compare(T a, T b, Comparator<? super T> c)` compares two objects using a comparator, returning `0` if the arguments are identical.

**Technical Definition:** "Returns `0` if the arguments are identical and `c.compare(a, b)` otherwise. Consequently, if both arguments are `null`, `0` is returned."

**Beginner-Friendly Explanation:** `Objects.compare()` is a null-safe way to compare two objects using a comparator. It handles the case where both objects are `null`.

### Purposes

- To provide null-safe comparison using a comparator
- To simplify comparison logic
- To avoid `NullPointerException` when comparing objects

### Syntax Rules and Structure

```java
public static <T> int compare(T a, T b, Comparator<? super T> c)
```

**Syntax Rules:**

- Returns `0` if `a` and `b` are both `null`.
- Returns `c.compare(a, b)` otherwise.
- Throws `NullPointerException` if `c` is `null` and `a` and `b` are not both `null`.

**Constraints and Limitations:**

- Requires a non-null comparator when at least one argument is non-null.

### Annotated Code Example

```java
// File: ObjectsCompareDemo.java
import java.util.Comparator;
import java.util.Objects;

public class ObjectsCompareDemo {
    public static void main(String[] args) {
        Comparator<String> comp = String::compareTo;

        System.out.println(Objects.compare("Apple", "Banana", comp));  // negative
        System.out.println(Objects.compare("Banana", "Apple", comp));  // positive
        System.out.println(Objects.compare(null, null, comp));         // 0
    }
}
```

**Expected Output:**
```
-1
1
0
```

**Step-by-Step Explanation:**

1. `Objects.compare("Apple", "Banana", comp)` uses `comp.compare("Apple", "Banana")`.
2. `Objects.compare(null, null, comp)` returns `0` because both are `null`.

### Real-World Cases

- **Sorting:** Null-safe sorting of collections.
- **Comparator Chaining:** Safe comparison in complex comparators.

---

## References Links

- Object (Java Platform SE 8) — Oracle - https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html
- Java Platform, Standard Edition Java API Reference — Class Object — Oracle - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html
- Objects (Java Development Kit Version 17 API Specification) — Oracle - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Objects.html
- Objects (Java Platform SE 8) — Oracle - https://docs.oracle.com/javase/8/docs/api/java/util/Objects.html
- MET13-J. Classes that define an equals() method must also define a hashCode() method — SEI CERT Oracle Coding Standard for Java - https://wiki.sei.cmu.edu/confluence/display/java/MET13-J.+Classes+that+define+an+equals%28%29+method+must+also+define+a+hashCode%28%29+method
- Object as a Superclass — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/objectclass.html
- JEP 421: Deprecate Finalization for Removal — OpenJDK - https://openjdk.org/jeps/421
- JSR 396: Java SE 21: Annex 2: class java.base/java.lang.Object — OpenJDK - https://cr.openjdk.org/~iris/se/21/latestSpec/apidiffs/java.base/java/lang/Object.html
- Object.java (JDK source) — OpenJDK - https://cr.openjdk.org/~darcy/4881419/src/share/classes/java/lang/Object.java.sdiff.html
- Objects Utility Class — Sip of Java (Inside.java) - https://inside.java/2023/05/28/sip-009/