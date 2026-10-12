# Java Inner Classes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An inner class is a class defined within the body of another class (the outer class). Inner classes are a form of nested class that is not explicitly or implicitly static, and they have a special relationship with their enclosing instance.

**Technical Definition:** Per the Java Language Specification, "An inner class is a nested class that is not explicitly or implicitly `static`. An inner class is one of the following: a member class that is not explicitly or implicitly static (§8.5), a local class that is not implicitly static (§14.3), or an anonymous class (§15.9.5)". The following nested classes are implicitly static, so are not inner classes: member enum classes, local enum classes, member record classes, local record classes, and member classes of an interface. "An inner class C is a direct inner class of a class or interface O if O is the immediately enclosing class or interface declaration of C and the declaration of C does not occur in a static context".

**Beginner-Friendly Explanation:** An inner class is a class defined inside another class, like a room inside a house. The inner class (room) has special access to the outer class's (house's) contents, even private ones. You can think of it as a "helper class" that is logically tied to the outer class and can access all its members.

### Key Characteristics

- **Enclosing Instance:** A non-static inner class is associated with an instance of its enclosing class and has direct access to that object's methods and fields.
- **Access to Private Members:** Inner classes have access to other members of the enclosing class, even if they are declared `private`.
- **No Static Members (Pre-Java 16):** Prior to Java SE 16, an inner class could not declare static initializers, and could only declare static members that were constant variables. Since Java 16, inner classes may declare and inherit static members and declare static initializers.
- **Instantiation Requires Outer Instance:** To instantiate a non-static inner class, you must first instantiate the outer class, then create the inner object using `outerObject.new InnerClass()`.
- **Types:** Inner classes include member classes (non-static), local classes, and anonymous classes.

### Prerequisites

- Basic understanding of Java classes and objects
- Familiarity with access modifiers (`private`, `protected`, `public`)
- Knowledge of the `static` keyword and its meaning
- Awareness of the `this` keyword and object references

### Related Programming Areas

- **Encapsulation:** Inner classes enhance encapsulation by hiding helper classes within the outer class.
- **Event Handling:** Anonymous inner classes are commonly used for event listeners in GUI programming.
- **Design Patterns:** The Iterator pattern, Builder pattern, and Strategy pattern often use inner classes.
- **Lambda Expressions:** Lambdas are a concise alternative to anonymous inner classes for functional interfaces.
- **Serialization:** Inner class serialization is strongly discouraged due to implicit outer references.

### Core Concepts / Features

---

## 1. Inner Classes

### Definitions

**Core Definition:** An inner class is a non-static nested class that is associated with an instance of its enclosing class.

**Technical Definition:** As defined by the JLS, an inner class is a nested class that is not explicitly or implicitly static. Inner classes are one of the following: a member class that is not explicitly or implicitly static, a local class that is not implicitly static, or an anonymous class. An inner class has direct access to the methods and fields of its enclosing instance.

**Beginner-Friendly Explanation:** An inner class is a class defined inside another class. Think of it as a "child class" that lives inside its "parent class" and can freely access all of the parent's stuff, even the private things.

### Purposes

- To logically group classes that are only used in one place, keeping them together
- To increase encapsulation by hiding helper classes from the outside world
- To enable a class to access the private members of its enclosing class
- To create more readable and maintainable code by placing code closer to where it is used

### Syntax Rules and Structure

**Complete General Syntax:**

```java
class OuterClass {
    // members of outer class
    class InnerClass {
        // members of inner class
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `OuterClass` | The enclosing class. |
| `InnerClass` | The inner class declared inside `OuterClass`. |
| No `static` | The absence of `static` makes it an inner (non-static) class. |

**Syntax Rules:**

- An inner class is declared inside the body of another class without the `static` modifier.
- An inner class can access all members (including private) of its enclosing class.
- An inner class cannot have static members prior to Java 16 (except constant variables).

**Constraints and Limitations:**

- An inner class instance cannot exist without an instance of the outer class.
- Inner classes cannot be instantiated from a static context without an outer instance.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Inner Class**

```java
// File: InnerClassDemo.java
public class InnerClassDemo {
    private String message = "Hello from outer";

    // Inner class (non-static)
    class Inner {
        public void display() {
            // Can access private member of outer class
            System.out.println(message);
        }
    }

    public static void main(String[] args) {
        // Step 1: Create outer instance
        InnerClassDemo outer = new InnerClassDemo();

        // Step 2: Create inner instance using outer instance
        InnerClassDemo.Inner inner = outer.new Inner();

        // Step 3: Call inner method
        inner.display();  // Hello from outer
    }
}
```

**Expected Output:**
```
Hello from outer
```

**Step-by-Step Explanation:**

1. `InnerClassDemo` is the outer class with a private field `message`.
2. `Inner` is the inner class with a method `display()` that accesses `message`.
3. In `main`, `outer` is created first.
4. `outer.new Inner()` creates an inner instance tied to `outer`.
5. `inner.display()` prints the outer class's private message.

### Real-World Cases

- **Iterator Pattern:** `ArrayList` uses a private inner class `Itr` to implement its iterator.
- **Event Handling:** GUI frameworks use inner classes for event listeners.
- **Builder Pattern:** Builders are often implemented as static nested or inner classes.

---

## 2. Non-static Nested Classes

### Definitions

**Core Definition:** A non-static nested class is an inner class that is associated with an instance of its enclosing class and has access to the enclosing instance's members.

**Technical Definition:** "Non-static nested classes are called inner classes. Nested classes that are declared static are called static nested classes". "Non-static nested classes (inner classes) have access to other members of the enclosing class, even if they are declared private". "An inner class is associated with an instance of its enclosing class and has direct access to that object's methods and fields".

**Beginner-Friendly Explanation:** A non-static nested class is a class that lives inside another class and is tied to a specific instance of that outer class. It can see and use everything in the outer class.

### Purposes

- To access the enclosing instance's non-public fields and methods directly
- To create a tightly coupled helper class that operates on the outer object's state
- To implement iterators and other objects that need access to the enclosing object's data

### Syntax Rules and Structure

```java
class Outer {
    class Inner {  // no static keyword
        // can access Outer's instance members
    }
}
```

**Syntax Rules:**

- No `static` modifier.
- Can access all members of the enclosing class.
- Must be instantiated through an outer instance: `outer.new Inner()`.

**Constraints and Limitations:**

- Cannot be instantiated from a static context without an outer instance.
- Holds an implicit reference to the enclosing instance, which can cause memory leaks if not managed carefully.

### Annotated Code Example

```java
// File: NonStaticNestedDemo.java
public class NonStaticNestedDemo {
    private int value = 100;

    class Inner {
        public void showValue() {
            System.out.println("Value: " + value);  // Accesses outer instance field
        }

        public void changeValue(int newValue) {
            value = newValue;  // Modifies outer instance field
        }
    }

    public static void main(String[] args) {
        NonStaticNestedDemo outer = new NonStaticNestedDemo();
        NonStaticNestedDemo.Inner inner = outer.new Inner();

        inner.showValue();       // Value: 100
        inner.changeValue(500);
        inner.showValue();       // Value: 500
        System.out.println(outer.value);  // 500
    }
}
```

**Expected Output:**
```
Value: 100
Value: 500
500
```

**Step-by-Step Explanation:**

1. `Inner` is a non-static nested class inside `NonStaticNestedDemo`.
2. `inner.showValue()` reads the outer class's `value` field.
3. `inner.changeValue(500)` modifies the outer class's `value` field.
4. The output confirms that the inner class has direct access to the outer instance's state.

### Real-World Cases

- **Custom Iterators:** `AbstractList.Itr` is a non-static inner class.
- **Event Listeners:** Anonymous inner classes for `ActionListener` are non-static.
- **Thread Classes:** `Thread` subclasses as inner classes capture the outer instance.

---

## 3. Static Nested Classes

### Definitions

**Core Definition:** A static nested class is a nested class declared with the `static` modifier, which means it does not have access to the enclosing class's instance members.

**Technical Definition:** "Nested classes that are declared static are called static nested classes". "Static nested classes do not have access to other members of the enclosing class". A static nested class is essentially a top-level class that has been nested for packaging convenience.

**Beginner-Friendly Explanation:** A static nested class is a class defined inside another class but with the `static` keyword. Unlike an inner class, it does not have a special relationship with an outer instance — it's just a class that happens to be defined inside another class.

### Purposes

- To group classes that are only used in one place without requiring an outer instance
- To avoid the implicit outer reference and potential memory leaks
- To provide a namespace for helper classes
- To enable instantiation without creating an outer class instance

### Syntax Rules and Structure

```java
class Outer {
    static class StaticNested {
        // cannot access Outer's instance members
    }
}
```

**Syntax Rules:**

- Declared with the `static` modifier.
- Can access static members of the outer class but not instance members.
- Instantiated like a top-level class: `Outer.StaticNested obj = new Outer.StaticNested();`

**Constraints and Limitations:**

- Cannot access instance fields or methods of the outer class directly.
- Does not have an implicit reference to an outer instance.

### Annotated Code Example

```java
// File: StaticNestedDemo.java
public class StaticNestedDemo {
    private static String staticMessage = "Static hello";
    private String instanceMessage = "Instance hello";

    static class StaticNested {
        public void display() {
            // Can access static members of outer class
            System.out.println(staticMessage);

            // Cannot access instanceMessage — compile error
            // System.out.println(instanceMessage);
        }
    }

    public static void main(String[] args) {
        // Instantiate without an outer instance
        StaticNestedDemo.StaticNested nested = new StaticNestedDemo.StaticNested();
        nested.display();  // Static hello
    }
}
```

**Expected Output:**
```
Static hello
```

**Step-by-Step Explanation:**

1. `StaticNested` is declared `static` inside `StaticNestedDemo`.
2. It can access `staticMessage` (a static field) but not `instanceMessage` (an instance field).
3. `new StaticNestedDemo.StaticNested()` creates an instance without needing an outer object.
4. `nested.display()` prints the static message.

### Real-World Cases

- **`Map.Entry`:** A static nested interface in the `Map` interface.
- **Builder Classes:** Static nested builder classes (e.g., `Person.Builder`).
- **Utility Classes:** Helper classes grouped inside a main class.

---

## 4. Local Classes

### Definitions

**Core Definition:** A local class is a class defined within a block (usually a method body) and is visible only within that block.

**Technical Definition:** "A local class is a class that is defined in a block of code, typically a method body". Local classes are not members of the enclosing class; they are local to the block in which they are declared. They can be static or non-static (if declared in a static context, they are implicitly static and thus not inner classes).

**Beginner-Friendly Explanation:** A local class is a class defined inside a method, like a temporary helper that only exists while that method is running.

### Purposes

- To create a class that is only needed within a specific method
- To access local variables and parameters of the enclosing method
- To define a named type when you need to create multiple instances within a method

### Syntax Rules and Structure

```java
void method() {
    class LocalClass {
        // can access effectively final local variables
    }
    LocalClass obj = new LocalClass();
}
```

**Syntax Rules:**

- Declared inside a method, constructor, or initializer block.
- Can access effectively final local variables and parameters of the enclosing block.
- Cannot be declared `public`, `private`, `protected`, or `static` (except in static contexts).

**Constraints and Limitations:**

- Cannot access non-final local variables (unless effectively final in Java 8+).
- Has the same scope as the block in which it is declared.

### Annotated Code Example

```java
// File: LocalClassDemo.java
public class LocalClassDemo {
    public static void main(String[] args) {
        final String greeting = "Hello from local";

        // Local class defined inside main method
        class LocalGreeter {
            public void greet() {
                System.out.println(greeting);
            }
        }

        LocalGreeter greeter = new LocalGreeter();
        greeter.greet();  // Hello from local
    }
}
```

**Expected Output:**
```
Hello from local
```

**Step-by-Step Explanation:**

1. `LocalGreeter` is defined inside the `main` method.
2. It accesses `greeting`, a local variable of `main`.
3. `LocalGreeter` is instantiated and used only within `main`.

### Real-World Cases

- **Validation Logic:** A local class for validating input within a method.
- **Algorithm Helpers:** A local class for temporary data structures during computation.
- **Event Handling:** Local classes for one-time event listeners.

---

## 5. Anonymous Classes

### Definitions

**Core Definition:** An anonymous class is a class without a name that is declared and instantiated in a single expression.

**Technical Definition:** "Anonymous classes enable you to make your code more concise. They enable you to declare and instantiate a class at the same time. They are like local classes except that they do not have a name". "While local classes are class declarations, anonymous classes are expressions, which means that you define the class in another expression".

**Beginner-Friendly Explanation:** An anonymous class is a class you create on the spot, without giving it a name, to use it immediately. It's like writing a quick note instead of creating a formal document.

### Purposes

- To create a one-time implementation of an interface or abstract class
- To reduce code verbosity when a class is used only once
- To encapsulate a single unit of behaviour to pass to other code
- To declare fields or additional methods in an inline implementation

### Syntax Rules and Structure

```java
InterfaceName obj = new InterfaceName() {
    // implementation
};
```

**Syntax Rules:**

- Declared as an expression: `new InterfaceName() { ... }`.
- Cannot have a name.
- Cannot be `abstract`.
- Can access effectively final local variables of the enclosing scope.

**Constraints and Limitations:**

- Cannot declare constructors (since there is no name).
- Cannot be `static`.
- Only one instance is created per expression.

### Annotated Code Example

```java
// File: AnonymousClassDemo.java
public class AnonymousClassDemo {
    interface Greeter {
        void greet(String name);
    }

    public static void main(String[] args) {
        // Anonymous class implementing the Greeter interface
        Greeter greeter = new Greeter() {
            @Override
            public void greet(String name) {
                System.out.println("Hello, " + name + "!");
            }
        };

        greeter.greet("Alice");  // Hello, Alice!
        greeter.greet("Bob");    // Hello, Bob!
    }
}
```

**Expected Output:**
```
Hello, Alice!
Hello, Bob!
```

**Step-by-Step Explanation:**

1. `Greeter` is an interface with one method.
2. The anonymous class implements `Greeter` inline.
3. `greeter` holds the anonymous class instance.
4. `greet("Alice")` invokes the anonymous implementation.

### Real-World Cases

- **Event Listeners:** `button.addActionListener(new ActionListener() { ... })`
- **Thread Creation:** `new Thread(new Runnable() { ... }).start()`
- **Comparator:** `Collections.sort(list, new Comparator<T>() { ... })`

---

## 6. Inner Class Access

### Definitions

**Core Definition:** Inner class access refers to the ability of an inner class to access the members of its enclosing class, and vice versa.

**Technical Definition:** "Non-static nested classes (inner classes) have access to other members of the enclosing class, even if they are declared private". The outer class can also access the members of its inner classes, subject to access modifiers.

**Beginner-Friendly Explanation:** Inner classes can see and use everything in the outer class, even private stuff. The outer class can also see the inner class's members, like members of any other class.

### Purposes

- To allow tight coupling between the inner and outer classes
- To enable the inner class to operate on the outer class's state
- To allow the outer class to use the inner class's functionality

### Syntax Rules and Structure

```java
class Outer {
    private int outerField = 10;

    class Inner {
        private int innerField = 20;

        void accessOuter() {
            System.out.println(outerField);  // Access outer private field
        }
    }

    void accessInner() {
        Inner inner = new Inner();
        System.out.println(inner.innerField);  // Access inner private field
    }
}
```

**Syntax Rules:**

- Inner classes can access all members of the outer class, including private ones.
- Outer classes can access all members of inner classes, including private ones.
- Access to local variables requires them to be effectively final.

**Constraints and Limitations:**

- The implicit outer reference can cause memory leaks if the inner class outlives the outer class.

### Annotated Code Example

```java
// File: InnerAccessDemo.java
public class InnerAccessDemo {
    private int outerValue = 100;

    class Inner {
        private int innerValue = 200;

        void display() {
            System.out.println("Outer: " + outerValue);  // Access outer private
            System.out.println("Inner: " + innerValue);
        }
    }

    void showInner() {
        Inner inner = new Inner();
        System.out.println("Accessing inner from outer: " + inner.innerValue);
    }

    public static void main(String[] args) {
        InnerAccessDemo outer = new InnerAccessDemo();
        InnerAccessDemo.Inner inner = outer.new Inner();

        inner.display();
        outer.showInner();
    }
}
```

**Expected Output:**
```
Outer: 100
Inner: 200
Accessing inner from outer: 200
```

**Step-by-Step Explanation:**

1. `Inner` accesses `outerValue` (private in outer class).
2. `showInner()` accesses `innerValue` (private in inner class).
3. Both directions of access are demonstrated.

### Real-World Cases

- **Iterator Implementation:** The inner iterator accesses the outer collection's private array.
- **State Machines:** Inner classes access the outer state machine's current state.
- **Builder Pattern:** The builder (inner class) accesses the outer class's private constructor.

---

## 7. Outer.this

### Definitions

**Core Definition:** `Outer.this` is a qualified `this` expression that refers to the enclosing instance of an inner class.

**Technical Definition:** "In an inner class constructor, the receiver is named `Outer.this`. It cannot be referred to as `this`". When an inner class needs to refer to its enclosing instance explicitly, it uses the syntax `OuterClassName.this`.

**Beginner-Friendly Explanation:** `Outer.this` is how an inner class says "the outer object that I belong to." It's like saying "my parent's this."

### Purposes

- To disambiguate between inner and outer members with the same name
- To pass the outer instance to other methods
- To access the outer instance explicitly when necessary

### Syntax Rules and Structure

```java
OuterClassName.this
```

**Syntax Rules:**

- Used within an inner class to refer to the enclosing instance.
- Can be used to access outer members or pass the outer instance.

**Constraints and Limitations:**

- Only available in non-static inner classes.
- Cannot be used in static nested classes (since they have no enclosing instance).

### Annotated Code Example

```java
// File: OuterThisDemo.java
public class OuterThisDemo {
    private String name = "Outer";

    class Inner {
        private String name = "Inner";

        void display() {
            System.out.println("Inner name: " + name);
            System.out.println("Outer name: " + OuterThisDemo.this.name);
        }
    }

    public static void main(String[] args) {
        OuterThisDemo outer = new OuterThisDemo();
        OuterThisDemo.Inner inner = outer.new Inner();
        inner.display();
    }
}
```

**Expected Output:**
```
Inner name: Inner
Outer name: Outer
```

**Step-by-Step Explanation:**

1. Both `OuterThisDemo` and `Inner` have a field named `name`.
2. Inside `Inner.display()`, `name` refers to the inner class's field.
3. `OuterThisDemo.this.name` explicitly refers to the outer class's field.

### Real-World Cases

- **Disambiguation:** Resolving name conflicts between inner and outer fields.
- **Callback Registration:** Passing `Outer.this` as a listener to another object.
- **Method Chaining:** Returning `Outer.this` from an inner class method.

---

## 8. Inner Class Limitations

### Definitions

**Core Definition:** Inner class limitations are the restrictions and potential pitfalls associated with using inner classes in Java.

**Technical Definition:** Inner classes have several limitations: they hold an implicit reference to the enclosing instance (which can cause memory leaks), serialization is error-prone, and they cannot be instantiated from a static context without an outer instance.

**Beginner-Friendly Explanation:** Inner classes are powerful but have some catches. They can cause memory leaks, are hard to serialize, and can't be used in static contexts easily.

### Purposes

- To understand the risks and costs of using inner classes
- To guide developers toward safer alternatives when appropriate

### Syntax Rules and Structure

**Key Limitations:**

| Limitation | Description |
|------------|-------------|
| Implicit outer reference | Every non-static inner class holds a reference to its outer instance, which can prevent garbage collection. |
| Serialization issues | Serializing inner classes is error-prone because the synthetic fields generated by the compiler are implementation-dependent. |
| No static members (pre-Java 16) | Before Java 16, inner classes could not declare static members (except constants). |
| Instantiation complexity | Requires an outer instance: `outer.new Inner()` |
| Cannot be used in static contexts | Inner classes cannot be instantiated from static methods without an outer instance. |

**Constraints and Limitations:**

- Memory leaks: A long-lived inner class instance (e.g., a listener) can keep the outer instance alive.
- Serialization: Should use static nested classes instead.

### Annotated Code Example

```java
// File: InnerLimitationsDemo.java
public class InnerLimitationsDemo {
    private byte[] largeData = new byte[1024 * 1024];  // 1 MB

    class Inner {
        void doSomething() {
            System.out.println("Doing something");
        }
    }

    public static void main(String[] args) {
        InnerLimitationsDemo outer = new InnerLimitationsDemo();
        Inner inner = outer.new Inner();

        // The inner instance holds a reference to outer,
        // preventing outer from being garbage collected.
        inner.doSomething();

        // Even after outer is no longer used, the inner reference
        // keeps the outer instance alive.
    }
}
```

**Expected Output:**
```
Doing something
```

**Step-by-Step Explanation:**

1. `Inner` holds an implicit reference to `outer`.
2. Even if `outer` is no longer referenced directly, `inner` keeps it alive.
3. This can lead to memory leaks if `inner` is stored in a long-lived collection.

### Real-World Cases

- **Listener Leaks:** Anonymous `ActionListener` inner classes can keep GUI frames alive.
- **Serialization Failures:** Inner classes with non-serializable outer classes cause `NotSerializableException`.
- **Static Context Errors:** Trying to instantiate an inner class from a static method without an outer instance.

---

## 9. Inner Class Best Practices

### Definitions

**Core Definition:** Inner class best practices are guidelines for using inner classes safely and effectively.

**Technical Definition:** Best practices derive from the SEI CERT Oracle Coding Standard and other authoritative sources. Key practices include: making inner classes `private` when possible, avoiding serialization of inner classes, preferring static nested classes when no outer instance access is needed, and being aware of memory leak risks.

**Beginner-Friendly Explanation:** Inner class best practices are the rules for using inner classes without causing problems like memory leaks or serialization issues.

### Purposes

- To prevent memory leaks and serialization problems
- To maintain clean encapsulation
- To choose the right type of nested class for the job
- To write maintainable and safe code

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Use `private` for inner classes | Declare inner classes and their methods/constructors as private to prevent external access. |
| Prefer static nested classes | If the nested class does not need access to the outer instance, make it static to avoid the implicit reference and memory leak risks. |
| Do not serialize inner classes | Use static nested classes if serialization is needed. |
| Be aware of memory leaks | Long-lived inner class instances (e.g., listeners) can keep the outer instance alive. |
| Use local or anonymous classes for one-time use | Use local classes if you need multiple instances; use anonymous classes for single-use implementations. |
| Document the enclosing relationship | Make it clear when a class is an inner class and depends on its outer instance. |

**Constraints and Limitations:**

- Inner classes create tight coupling between the inner and outer classes.
- Overuse of inner classes can make code harder to navigate.

### Annotated Code Example

```java
// File: BestPracticesInner.java
public class BestPracticesInner {
    private int data = 42;

    // GOOD: private inner class
    private class Inner {
        private void display() {
            System.out.println("Data: " + data);
        }
    }

    // GOOD: static nested class when no outer access needed
    private static class StaticHelper {
        static void help() {
            System.out.println("Helping without outer instance");
        }
    }

    public void useInner() {
        Inner inner = new Inner();
        inner.display();
    }

    public static void main(String[] args) {
        BestPracticesInner obj = new BestPracticesInner();
        obj.useInner();           // Data: 42
        StaticHelper.help();      // Helping without outer instance
    }
}
```

**Expected Output:**
```
Data: 42
Helping without outer instance
```

**Step-by-Step Explanation:**

1. `Inner` is `private`, so it cannot be accessed from outside the class.
2. `StaticHelper` is `static`, so it doesn't hold an outer reference.
3. `useInner()` creates and uses the inner class internally.
4. `StaticHelper.help()` is called without an outer instance.

### Real-World Cases

- **`ArrayList.Itr`:** Private inner class for iteration.
- **`HashMap.Node`:** Static nested class for entries.
- **Builder Pattern:** Static nested `Builder` class in immutable classes.

---

## References Links

- Nested Classes — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html
- Java Language Specification, §8.1.3: Inner Classes and Enclosing Instances - https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.1.3
- When to Use Nested Classes, Local Classes, Anonymous Classes, and Lambda Expressions — Oracle - https://docs.oracle.com/javase/tutorial/java/javaOO/whentouse.html
- Local Classes — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html
- Anonymous Classes — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html
- SEI CERT Oracle Coding Standard — SCP02-J: Use inner classes carefully - https://wiki.sei.cmu.edu/confluence/display/java/SCP02-J.+Use+inner+classes+carefully
- SEI CERT Oracle Coding Standard — SER33-J: Do not serialize instances of inner classes - https://wiki.sei.cmu.edu/confluence/display/java/SER33-J.+Do+not+serialize+instances+of+inner+classes
- JDK Bug: Omit enclosing instance fields from inner classes that don't use it — OpenJDK - https://bugs.openjdk.org/browse/JDK-8271623
- Inner class may be 'static' — JetBrains Inspectopedia - https://www.jetbrains.com/help/inspectopedia/InnerClassMayBeStatic.html