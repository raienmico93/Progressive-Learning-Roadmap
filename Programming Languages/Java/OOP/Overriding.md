# Java Method Overriding: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Method overriding** in Java is a mechanism where a subclass provides a specific implementation for a method that is already defined in its superclass. The overriding method has the same name, parameter list, and return type (or a covariant subtype) as the method it overrides.

### Technical Definition

An instance method in a subclass overrides an instance method in its superclass when it has the same signature (name and parameter types) and a return type that is return-type-substitutable for the overridden method. At runtime, the Java Virtual Machine uses **dynamic method dispatch** to select the most specific implementation based on the actual class of the object, not the declared type of the reference . Overriding is a core mechanism for runtime polymorphism and the "Is-A" relationship in object-oriented programming.

### Beginner-Friendly Explanation

Imagine a family recipe book. Your grandmother wrote down a recipe for "cake." Your mother inherited the book but decided her cake should use chocolate instead of vanilla. She **overrides** the original recipe with her own version. When anyone in the family asks for "the cake recipe," they get the most specific version available—your mother's, not the original. In Java, if a subclass defines its own version of a method inherited from its parent, the subclass's version is used.

### Key Characteristics

- **Runtime resolution**: The method to invoke is determined at runtime based on the actual object type, not the reference type.
- **Signature matching**: The overriding method must have the same name and parameter types.
- **Covariant returns**: The return type may be a subtype of the overridden method's return type.
- **Access expansion only**: The overriding method cannot reduce visibility.
- **Exception restriction**: The overriding method cannot throw broader checked exceptions.
- **`@Override` annotation**: A compile-time safety net that verifies the method actually overrides something.

### Prerequisites

- Basic understanding of classes, inheritance, and the `extends` keyword.
- Familiarity with method signatures (name + parameter types).
- Knowledge of access modifiers (`public`, `protected`, package-private, `private`).
- Understanding of checked vs. unchecked exceptions.

### Related Programming Areas

- **Polymorphism**: Overriding is the mechanism for runtime polymorphism.
- **Dynamic Dispatch**: The JVM's runtime method selection process.
- **Design Patterns**: Template Method, Strategy, and Factory patterns rely on overriding.
- **Liskov Substitution Principle**: Overriding must preserve the superclass contract.

### Core Concepts / Features

1. Runtime Method Dispatch
2. Override Rules (Signatures Must Match Exactly)
3. `@Override` Annotation Utility
4. Covariant Return Types
5. Access-Level Constraints
6. Exception Handling Constraints in Overriding


## Core Concept 1: Runtime Method Dispatch

### Definitions

**Core Definition**: **Runtime method dispatch** (also called dynamic method lookup or late binding) is the process by which the JVM determines which method implementation to invoke based on the actual runtime type of the object.

**Technical Definition**: When an instance method is invoked, the Java compiler determines the **signature** of the method to invoke based on the compile-time types of the arguments . However, the **actual implementation** that executes is selected at runtime using the runtime class of the object on which the method is invoked. This is known as **dynamic method lookup** . A cast to a supertype does not change the method invoked; only the actual object's class matters .

**Beginner-Friendly Explanation**: Imagine you call a restaurant and ask for "the special." The person who answers (the reference type) might be the manager, but the kitchen (the actual object) decides what the special actually is. The manager doesn't cook it—the kitchen does. In Java, the reference type doesn't determine which method runs; the actual object does.

### Purposes

- To enable polymorphic behavior where a single reference type can invoke different implementations.
- To allow subclasses to customize inherited behavior without changing the calling code.
- To support the Open/Closed Principle (open for extension, closed for modification).
- To make code more flexible and extensible through inheritance hierarchies.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Superclass
class Superclass {
    public void methodName() { /* base implementation */ }
}

// Subclass overrides
class Subclass extends Superclass {
    @Override
    public void methodName() { /* specialized implementation */ }
}

// Runtime dispatch
Superclass ref = new Subclass();
ref.methodName(); // Invokes Subclass.methodName()
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Superclass ref` | The declared (compile-time) type of the reference. |
| `new Subclass()` | The actual (runtime) type of the object. |
| `ref.methodName()` | The compiler checks that `methodName()` exists in `Superclass`; the JVM invokes the most specific version. |

**Syntax Rules:**

- The compiler verifies the method exists in the reference type.
- The JVM selects the method based on the runtime class of the object.
- Casts to supertypes do not change dispatch; only the object's actual class matters .
- `super.methodName()` bypasses overriding and invokes the superclass's version .

**Constraints and Limitations:**

- Static methods are not dispatched dynamically (they use static binding).
- Private methods cannot be overridden, so no dynamic dispatch occurs.
- Fields are not polymorphic; field access uses the reference type, not the runtime type .

### Annotated Complete Code Examples

**Example 1: Runtime Method Dispatch**

```java
/**
 * Demonstrates runtime method dispatch.
 */
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Cat meows");
    }
}

public class DispatchDemo {
    public static void main(String[] args) {
        Animal[] animals = { new Dog(), new Cat(), new Animal() };

        for (Animal a : animals) {
            a.makeSound(); // Runtime dispatch selects the actual type's method
        }

        // Casting does NOT change dispatch
        Animal ref = new Dog();
        ref.makeSound(); // Still "Dog barks"

        // super bypasses overriding
        // (inside Dog, super.makeSound() would call Animal's version)
    }
}
```

**Expected Output:**

```
Dog barks
Cat meows
Animal makes a sound
Dog barks
```

**Why This Output Occurs:**
- The array `Animal[]` holds objects of different runtime types.
- Each `a.makeSound()` call uses the **runtime class** of the object (`Dog`, `Cat`, or `Animal`) to select the implementation .
- The cast `Animal ref = new Dog()` does not change the runtime type; `ref.makeSound()` still invokes `Dog`'s version .

**Step-by-Step Setup Guide:**
1. Create `DispatchDemo.java` with the classes above.
2. Compile with `javac DispatchDemo.java`.
3. Run with `java DispatchDemo`.
4. Observe the output.

### Real-World Cases

- **Collections Framework**: `List` references invoke `ArrayList`'s or `LinkedList`'s implementations.
- **GUI frameworks**: A `Button` reference invokes the overridden `paint()` method of a custom button.
- **Template Method Pattern**: The superclass defines the algorithm; subclasses override specific steps.

### References

- Java Language Specification – Dynamic Method Lookup - https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.12.4 
- Java Language Specification – Method Invocation using super - https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html 


## Core Concept 2: Override Rules (Signatures Must Match Exactly)

### Definitions

**Core Definition**: For a method to override another, it must have the **same signature**—the same name and the same number and types of parameters—as the method it overrides.

**Technical Definition**: The signature of a method consists of its name and the number and types of its formal parameters. An overriding method must have the same signature as the overridden method . The return type must be return-type-substitutable (see Covariant Return Types). The `throws` clause must not introduce broader checked exceptions (see Exception Handling Constraints). The access modifier must not be more restrictive (see Access-Level Constraints).

**Beginner-Friendly Explanation**: The method signature is like a fingerprint. Two methods override each other only if their fingerprints match: same name, same parameters. If you change the name or the parameters, you're not overriding—you're overloading.

### Purposes

- To ensure the subclass method is correctly recognized as an override by the compiler.
- To enable polymorphic dispatch based on a consistent method contract.
- To prevent accidental overloads that don't actually override.
- To allow the compiler to verify the override relationship (with `@Override`).

### Syntax Rules and Structure

**Signature Comparison:**

| Element | Must Match? |
|---------|-------------|
| Method name | Yes |
| Parameter types (order, number) | Yes |
| Parameter names | No (cosmetic only) |
| Return type | Must be covariant (subtype) |
| Access modifier | Must be same or wider |
| `throws` clause | Must not be broader (checked only) |

**Complete General Syntax:**

```java
class Superclass {
    public void methodName(int param) { }
}

class Subclass extends Superclass {
    @Override
    public void methodName(int param) { } // Same signature — valid override
}
```

**Syntax Rules:**

- The method name must be identical.
- The parameter list must have the same number, types, and order.
- Generic type parameters may differ in name but must have compatible bounds.
- The return type must be a subtype (covariant) or identical.
- The `@Override` annotation verifies the override relationship at compile time .

**Constraints and Limitations:**

- Changing the parameter list results in **overloading**, not overriding.
- Changing the return type to an unrelated type causes a compile-time error.
- Private methods cannot be overridden (they are not inherited).
- Static methods cannot be overridden (they are hidden).

### Annotated Complete Code Examples

**Example 1: Signature Matching**

```java
/**
 * Demonstrates signature requirements for overriding.
 */
class Parent {
    public void display(String message) {
        System.out.println("Parent: " + message);
    }
}

class Child extends Parent {
    @Override
    public void display(String message) { // Same signature
        System.out.println("Child: " + message);
    }

    // This is OVERLOADING, not overriding (different parameter type)
    public void display(int number) {
        System.out.println("Child number: " + number);
    }
}

public class SignatureDemo {
    public static void main(String[] args) {
        Parent p = new Child();
        p.display("Hello"); // Overridden method — dispatches to Child

        Child c = new Child();
        c.display(42); // Overloaded method — resolved at compile time
    }
}
```

**Expected Output:**

```
Child: Hello
Child number: 42
```

**Why This Output Occurs:**
- `Child.display(String)` has the same signature as `Parent.display(String)`, so it overrides.
- `p.display("Hello")` dispatches to `Child`'s version at runtime.
- `Child.display(int)` is an **overload** (different parameter type), resolved at compile time.

### Real-World Cases

- **`toString()`**: Overriding `Object.toString()` with the same signature.
- **`equals(Object)`**: Overriding `Object.equals()` with the same signature.
- **`compareTo()`**: Implementing `Comparable<T>.compareTo()` with the correct signature.

### References

- Oracle Java Tutorials – Overriding and Hiding Methods - https://docs.oracle.com/javase/tutorial/java/IandI/override.html 
- Java Language Specification – Method Signatures - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html 


## Core Concept 3: `@Override` Annotation Utility

### Definitions

**Core Definition**: The **`@Override`** annotation is a marker that indicates a method is intended to override a method in a supertype. The compiler generates an error if the annotated method does not actually override or implement a method.

**Technical Definition**: `@Override` is a source-level annotation (`@Retention(SOURCE)`) that instructs the compiler to verify that the annotated method overrides or implements a method declared in a supertype (including interface methods) . If the method does not override anything—for example, due to a typo in the method name or a mismatched parameter list—the compiler reports an error .

**Beginner-Friendly Explanation**: The `@Override` annotation is like a spell-checker for overriding. If you think you're overriding a method but accidentally misspell the name or use the wrong parameters, the compiler catches it immediately instead of silently treating it as a new method.

### Purposes

- To catch typos and signature mismatches at compile time.
- To document the programmer's intent that the method overrides a superclass method.
- To improve code readability by making the override relationship explicit.
- To ensure that interface methods are correctly implemented.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
@Override
public ReturnType methodName(Parameters) { }
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@Override` | The annotation placed before the method declaration. |
| `public ReturnType` | The method's access modifier and return type. |
| `methodName(Parameters)` | The method signature that must match the overridden method. |

**Syntax Rules:**

- `@Override` can be applied only to methods .
- The method must actually override or implement a method from a supertype.
- The annotation is optional but strongly recommended .
- The annotation has source-level retention; it is not present in compiled bytecode .

**Constraints and Limitations:**

- Using `@Override` on a method that does not override anything causes a compile-time error.
- The annotation cannot be used on fields or constructors.
- Some IDEs and compilers may warn if `@Override` is missing on an overriding method.

### Annotated Complete Code Examples

**Example 1: `@Override` Catching a Typo**

```java
/**
 * Demonstrates @Override catching a typo.
 */
class Base {
    public void display() {
        System.out.println("Base display");
    }
}

class Derived extends Base {
    @Override
    public void display() { // Correct — compiles
        System.out.println("Derived display");
    }

    // @Override
    // public void dispaly() { } // Compile error: does not override anything
}

public class OverrideAnnotationDemo {
    public static void main(String[] args) {
        Base b = new Derived();
        b.display();
    }
}
```

**Expected Output:**

```
Derived display
```

**Why This Output Occurs:**
- `Derived.display()` correctly overrides `Base.display()`.
- The commented-out `@Override public void dispaly()` would cause a compile error because `dispaly` (misspelled) does not exist in `Base` .

### Real-World Cases

- **IDE warnings**: Modern IDEs flag missing `@Override` annotations as a code quality issue.
- **Interface implementation**: `@Override` confirms that a method implements an interface method.
- **Code reviews**: The annotation makes the override relationship immediately visible.

### References

- Oracle Java API – Annotation Type Override - https://docs.oracle.com/javase/8/docs/api/java/lang/Override.html 
- Apache JIRA – @Override should be used on overriding and implementing methods - https://issues.apache.org/jira/browse/ROL-2159 
- Cornell – Annotations in Java - https://www.cs.cornell.edu/courses/JavaAndDS/files/annotation.pdf 


## Core Concept 4: Covariant Return Types

### Definitions

**Core Definition**: A **covariant return type** allows an overriding method to return a **subtype** of the return type declared in the overridden method.

**Technical Definition**: If a method declaration `d1` with return type `R1` overrides a method `d2` with return type `R2`, then `R1` must be **return-type-substitutable** for `R2`. This allows covariant return types—refining the return type when overriding . The covariant return type feature was introduced in Java SE 5.0 .

**Beginner-Friendly Explanation**: If the parent's method returns "a fruit," the child's override can return "an apple" because an apple is a fruit. The child can be more specific about what it returns, but not less specific.

### Purposes

- To allow overriding methods to return more specific types without breaking polymorphism.
- To reduce the need for casting when using subclass-specific return types.
- To make APIs more expressive and type-safe.
- To support fluent interfaces and builder patterns.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
class Superclass {
    public Superclass self() { return this; }
}

class Subclass extends Superclass {
    @Override
    public Subclass self() { return this; } // Covariant return type
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Superclass self()` | The superclass method's return type. |
| `Subclass self()` | The subclass's override returns a subtype. |

**Syntax Rules:**

- The return type of the overriding method must be a subtype of the overridden method's return type .
- If the return type is not a subtype, a compile-time unchecked warning occurs (or error in some contexts) .
- Covariant returns work with classes and interfaces.
- The return type must be a reference type (not primitive).

**Constraints and Limitations:**

- Primitives cannot be covariant (e.g., `int` and `long` are unrelated).
- The subtype relationship must be genuine (inheritance or implementation).
- Generic type erasure can introduce complications .

### Annotated Complete Code Examples

**Example 1: Covariant Return Type**

```java
/**
 * Demonstrates covariant return types.
 */
class Shape {
    public Shape copy() {
        System.out.println("Shape.copy()");
        return new Shape();
    }
}

class Circle extends Shape {
    @Override
    public Circle copy() { // Covariant return type
        System.out.println("Circle.copy()");
        return new Circle();
    }
}

public class CovariantDemo {
    public static void main(String[] args) {
        Shape s = new Circle();
        Shape copy = s.copy(); // Runtime dispatch to Circle.copy()
        System.out.println("Copied: " + copy.getClass().getSimpleName());
    }
}
```

**Expected Output:**

```
Circle.copy()
Copied: Circle
```

**Why This Output Occurs:**
- `Circle.copy()` overrides `Shape.copy()` with a covariant return type (`Circle` is a subtype of `Shape`).
- At runtime, `s.copy()` dispatches to `Circle.copy()`.
- The returned object is a `Circle`, but it is assigned to a `Shape` reference (valid because `Circle` is a `Shape`).

### Real-World Cases

- **`clone()`**: Covariant return types allow `clone()` to return the specific type.
- **Builder patterns**: Fluent builders return the concrete builder type.
- **Factory methods**: Subclasses can return more specific product types.

### References

- Java Language Specification – Covariant Return Types - https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.4.5 
- Oracle Java Tutorials – Overriding and Hiding Methods - https://docs.oracle.com/javase/tutorial/java/IandI/override.html 


## Core Concept 5: Access-Level Constraints

### Definitions

**Core Definition**: An overriding method **cannot reduce the visibility** of the method it overrides. It can only maintain or increase the access level.

**Technical Definition**: The access modifier of an overriding or hiding method must provide **at least as much access** as the overridden or hidden method . Specifically: `public` → must be `public`; `protected` → must be `protected` or `public`; package-private → must not be `private` .

**Beginner-Friendly Explanation**: If a parent makes a method available to "family only" (protected), the child cannot make it "private" (only me). The child can open it up to "everyone" (public) or keep it "family only" (protected), but cannot close it down.

### Purposes

- To preserve the Liskov Substitution Principle (subclasses must honor superclass contracts).
- To ensure polymorphic code works correctly (a superclass reference can still access the method).
- To prevent subclasses from breaking client expectations.
- To maintain API compatibility when extending classes.

### Syntax Rules and Structure

**Access Expansion Rules:**

| Overridden Method | Overriding Method Allowed |
|-------------------|---------------------------|
| `public` | `public` only |
| `protected` | `protected` or `public` |
| package-private | package-private, `protected`, or `public` |
| `private` | Cannot be overridden |

**Complete General Syntax:**

```java
class Parent {
    protected void method() { }
}

class Child extends Parent {
    @Override
    public void method() { } // Valid: increased access
}

// class BadChild extends Parent {
//     @Override
//     private void method() { } // Compile error: reduced access
// }
```

**Syntax Rules:**

- The overriding method's access modifier must be the same or wider.
- `private` methods are not inherited and cannot be overridden.
- `final` methods cannot be overridden regardless of access.
- The rule applies to methods and nested classes but not fields .

**Constraints and Limitations:**

- Reducing access causes a compile-time error.
- Package-private methods can be overridden only within the same package (if the subclass is in a different package, it cannot override them).

### Annotated Complete Code Examples

**Example 1: Access-Level Constraints**

```java
/**
 * Demonstrates access-level rules for overriding.
 */
class Parent {
    protected void display() {
        System.out.println("Parent display");
    }
}

class Child extends Parent {
    @Override
    public void display() { // Increased access: protected → public
        System.out.println("Child display");
    }
}

// class BadChild extends Parent {
//     @Override
//     private void display() { } // Compile error: cannot reduce access
// }

public class AccessDemo {
    public static void main(String[] args) {
        Parent p = new Child();
        p.display(); // Valid: display() is accessible via Parent
    }
}
```

**Expected Output:**

```
Child display
```

**Why This Output Occurs:**
- `Child.display()` overrides `Parent.display()` with increased access (`protected` → `public`).
- `p.display()` is valid because `display()` is `protected` in `Parent` and accessible.
- Reducing access to `private` would cause a compile-time error .

### Real-World Cases

- **API evolution**: Subclasses can widen access but not narrow it.
- **Framework extension**: Overridden framework methods must maintain at least the original visibility.
- **Interface implementation**: Interface methods are implicitly `public`, so implementations must be `public`.

### References

- Java Language Specification – Requirements in Overriding and Hiding - https://docs.oracle.com/javase/specs/jls/se23/jls23.pdf 
- Java Language Specification – Class Declarations - https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html 


## Core Concept 6: Exception Handling Constraints in Overriding

### Definitions

**Core Definition**: An overriding method **cannot throw broader checked exceptions** than the method it overrides. It may throw the same exceptions, narrower (subtype) exceptions, or no exceptions at all.

**Technical Definition**: A method that overrides or hides another method may not be declared to throw more checked exceptions than the overridden or hidden method . For every checked exception in the overriding method's `throws` clause, the same exception class or one of its supertypes must occur in the overridden method's `throws` clause . This rule does not apply to unchecked exceptions (`RuntimeException` and `Error`), which can be thrown freely .

**Beginner-Friendly Explanation**: If the parent's method says "I might throw a `FileNotFoundException`," the child cannot say "I might throw a `IOException`" (which is broader). The child can say "I might throw a `FileNotFoundException`" (same) or "I might throw a `FileSystemException`" (narrower), or nothing at all. Unchecked exceptions are not restricted.

### Purposes

- To preserve the contract of the superclass method (clients can handle the same exceptions).
- To prevent overriding methods from surprising callers with unexpected checked exceptions.
- To maintain substitutability (Liskov Substitution Principle).
- To ensure polymorphic code can handle exceptions correctly.

### Syntax Rules and Structure

**Exception Rules:**

| Overridden `throws` | Overriding `throws` Allowed |
|---------------------|------------------------------|
| No checked exceptions | No checked exceptions (unchecked OK) |
| `IOException` | `IOException`, `FileNotFoundException`, or none |
| `Exception` | Any checked exception (since `Exception` is the root) |

**Complete General Syntax:**

```java
class Parent {
    public void method() throws IOException { }
}

class Child extends Parent {
    @Override
    public void method() throws FileNotFoundException { } // Narrower — OK
}

// class BadChild extends Parent {
//     @Override
//     public void method() throws Exception { } // Broader — compile error
// }
```

**Syntax Rules:**

- The overriding method cannot declare checked exceptions that are not subtypes of exceptions declared by the overridden method .
- Unchecked exceptions (`RuntimeException` and its subclasses, `Error` and its subclasses) are not restricted .
- If the overridden method declares no checked exceptions, the overriding method cannot declare any checked exceptions .
- If the overridden method is in an interface, the same rules apply.

**Constraints and Limitations:**

- Throwing a broader checked exception causes a compile-time error.
- The rule applies only to checked exceptions; unchecked exceptions are unrestricted.
- Generic type erasure can introduce unchecked warnings .

### Annotated Complete Code Examples

**Example 1: Exception Constraints**

```java
import java.io.*;

/**
 * Demonstrates exception handling constraints in overriding.
 */
class Parent {
    public void readFile() throws IOException {
        System.out.println("Parent reads file");
    }
}

class Child extends Parent {
    @Override
    public void readFile() throws FileNotFoundException { // Narrower — OK
        System.out.println("Child reads file");
        throw new FileNotFoundException("Test exception");
    }
}

// class BadChild extends Parent {
//     @Override
//     public void readFile() throws Exception { } // Broader — compile error
// }

public class ExceptionOverrideDemo {
    public static void main(String[] args) {
        Parent p = new Child();
        try {
            p.readFile();
        } catch (FileNotFoundException e) {
            System.out.println("Caught: " + e.getMessage());
        } catch (IOException e) {
            System.out.println("Caught broader: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Child reads file
Caught: Test exception
```

**Why This Output Occurs:**
- `Child.readFile()` declares `FileNotFoundException`, which is a subtype of `IOException` — valid narrowing .
- The `catch (FileNotFoundException)` block handles the specific exception.
- Declaring `throws Exception` would cause a compile error because `Exception` is broader than `IOException` .

### Real-World Cases

- **I/O operations**: Subclasses can narrow `IOException` to `FileNotFoundException`.
- **Framework methods**: Overriding methods must respect the superclass's exception contract.
- **Interface implementations**: Implementations cannot add checked exceptions not declared in the interface.

### References

- Java Language Specification – Requirements in Overriding and Hiding - https://docs.oracle.com/javase/specs/jls/se23/jls23.pdf 
- NIELIT – Exception Handling with Method Overriding - http://nielit.gov.in/gorakhpur/sites/default/files/Gorakhpur/alevel_2_java_20APR_AB.pdf 


## References

- Java Language Specification – Requirements in Overriding and Hiding - https://docs.oracle.com/javase/specs/jls/se23/jls23.pdf 
- Oracle Java Tutorials – Overriding and Hiding Methods - https://docs.oracle.com/javase/tutorial/java/IandI/override.html 
- Oracle Java API – Annotation Type Override - https://docs.oracle.com/javase/8/docs/api/java/lang/Override.html 
- Java Language Specification – Dynamic Method Lookup - https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html 
- Java Language Specification – Method Invocation using super - https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html 
- NIELIT – Exception Handling with Method Overriding - http://nielit.gov.in/gorakhpur/sites/default/files/Gorakhpur/alevel_2_java_20APR_AB.pdf 
- Java Language Specification – Covariant Return Types - https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html 
- Apache JIRA – @Override should be used on overriding and implementing methods - https://issues.apache.org/jira/browse/ROL-2159 
- Cornell – Annotations in Java - https://www.cs.cornell.edu/courses/JavaAndDS/files/annotation.pdf 
- Carnegie Mellon SEI – MET02-J: Avoid ambiguous uses of overloading - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88876742 
- MIT – Method Invocation using super - https://groups.csail.mit.edu/medg/courses/6001-java/15.doc.html 
- Cambridge University Press – Advanced Java Features (Dynamic Method Dispatch) - https://www.cambridge.org/core/books/abs/short-course-in-computational-science-and-engineering/advanced-java-features/FA3CD02DEF4332EE7B32013D64F131CE 