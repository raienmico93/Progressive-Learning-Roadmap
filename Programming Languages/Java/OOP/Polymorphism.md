# Java Polymorphism: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Polymorphism is a fundamental principle of object-oriented programming that allows objects of different classes to be treated as objects of a common superclass, with the specific behaviour determined at runtime based on the actual object type.

**Technical Definition:** Polymorphism (from Greek: "poly" = many, "morph" = form) is the ability of a reference variable to refer to objects of different types and to invoke methods that behave differently depending on the actual type of the object. In Java, polymorphism is achieved through method overriding (runtime polymorphism) and method overloading (compile-time polymorphism). The Java Virtual Machine uses dynamic method dispatch to determine which method implementation to invoke at runtime based on the actual class of the object, not the declared type of the reference.

**Beginner-Friendly Explanation:** Polymorphism is like a universal remote control. You press the "play" button, and whether it's a DVD player, a streaming box, or a game console, the "play" command works — but each device does something different. In Java, you can write code that says "call the `makeSound()` method" on an `Animal` reference, and whether it's actually a `Dog` or a `Cat`, the correct sound is produced.

### Key Characteristics

- **"Many Forms":** The same method call can behave differently depending on the actual object type.
- **Runtime Resolution:** The specific method implementation is determined at runtime, not compile time (for overriding).
- **Substitutability:** A subclass object can be used wherever a superclass object is expected (Liskov Substitution Principle).
- **Interface-Based:** Polymorphism works through inheritance hierarchies and interfaces.
- **Dynamic Dispatch:** The JVM uses the actual object's class to select the method implementation.
- **Two Types:** Compile-time polymorphism (overloading) and runtime polymorphism (overriding).

### Prerequisites

- Understanding of classes, objects, and inheritance
- Familiarity with method declaration and invocation
- Knowledge of method overriding and the `@Override` annotation
- Awareness of the `super` keyword and constructor chaining
- Basic understanding of the class hierarchy and `Object` as the root

### Related Programming Areas

- **Object-Oriented Programming (OOP):** Polymorphism is one of the four fundamental OOP principles (along with encapsulation, inheritance, and abstraction).
- **Design Patterns:** Many patterns (Strategy, Factory, Observer, Command) rely heavily on polymorphism.
- **Frameworks:** Frameworks like Spring and Hibernate use polymorphism extensively for dependency injection and ORM.
- **Collections Framework:** The `Collection`, `List`, `Set`, and `Map` interfaces demonstrate polymorphic behaviour.
- **Functional Programming:** Java 8+ lambdas and method references provide a form of polymorphic behaviour.

### Core Concepts / Features

---

## 1. Polymorphism

### Definitions

**Core Definition:** Polymorphism is the ability of an object to take on many forms, allowing a single interface to represent different underlying types.

**Technical Definition:** In Java, polymorphism enables a reference variable of a superclass type to refer to an object of any subclass type. When a method is invoked on that reference, the JVM determines which implementation to execute based on the actual type of the object at runtime. This is known as dynamic method dispatch. Polymorphism is implemented through method overriding (runtime) and method overloading (compile-time).

**Beginner-Friendly Explanation:** Polymorphism means "many shapes." In Java, it means you can use a general type (like `Animal`) to refer to specific types (like `Dog` or `Cat`), and when you call a method, the specific type's version runs. It's like saying "give me a vehicle" and getting either a car or a bike — both can `move()`, but they do it differently.

### Purposes

- To enable writing flexible, reusable code that works with objects of multiple types
- To allow a single method or data structure to handle different types uniformly
- To support the "open-closed principle" by allowing new types to be added without modifying existing code
- To reduce coupling between components by programming to interfaces or abstract classes
- To enable frameworks and libraries to work with user-defined types through common contracts

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Superclass or interface reference
SuperType reference = new SubType();

// Polymorphic method call
reference.methodName();  // Resolved at runtime
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `SuperType` | The declared type of the reference (class or interface). |
| `SubType` | The actual type of the object being created. |
| `reference` | A variable that can hold any subtype of `SuperType`. |
| `methodName()` | A method defined in `SuperType` and overridden in `SubType`. |

**Syntax Rules:**

- The reference type determines which methods can be called (compile-time check).
- The object type determines which method implementation runs (runtime dispatch).
- Polymorphism works with both classes (inheritance) and interfaces (implementation).

**Constraints and Limitations:**

- Polymorphism does not apply to fields (fields are resolved at compile time based on the reference type).
- Static methods are not polymorphic; they are resolved at compile time.
- Private methods are not polymorphic; they are not inherited.
- Final methods cannot be overridden, so they are not polymorphic.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Polymorphism with Classes**

```java
// File: PolymorphismDemo.java
// Superclass
class Animal {
    public void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

// Subclass
class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof! Woof!");
    }
}

// Subclass
class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

public class PolymorphismDemo {
    public static void main(String[] args) {
        // Polymorphic references: SuperType reference = new SubType()
        Animal myDog = new Dog();   // Upcasting
        Animal myCat = new Cat();   // Upcasting

        // Polymorphic method calls
        myDog.makeSound();  // Resolves to Dog's implementation
        myCat.makeSound();  // Resolves to Cat's implementation
    }
}
```

**Expected Output:**
```
Woof! Woof!
Meow!
```

**Step-by-Step Explanation:**

1. `Animal` is the superclass with a generic `makeSound()` method.
2. `Dog` and `Cat` are subclasses that override `makeSound()`.
3. `Animal myDog = new Dog()` creates a `Dog` object but assigns it to an `Animal` reference (upcasting).
4. When `myDog.makeSound()` is called, the JVM checks the actual object type (`Dog`) and invokes `Dog`'s version of the method.
5. Similarly, `myCat.makeSound()` invokes `Cat`'s version.
6. This demonstrates runtime polymorphism — the same method call produces different behaviour based on the object type.

**Example 2: Polymorphism with Arrays**

```java
// File: PolymorphismArrayDemo.java
class Shape {
    public double area() {
        return 0;
    }
}

class Circle extends Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

class Rectangle extends Shape {
    private double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }
}

public class PolymorphismArrayDemo {
    public static void main(String[] args) {
        // Array of superclass type holding subclass objects
        Shape[] shapes = {
            new Circle(5),
            new Rectangle(4, 3),
            new Circle(2)
        };

        // Polymorphic method calls on each element
        for (Shape shape : shapes) {
            System.out.println("Area: " + shape.area());
        }
    }
}
```

**Expected Output:**
```
Area: 78.53981633974483
Area: 12.0
Area: 12.566370614359172
```

**Step-by-Step Explanation:**

1. `Shape` is the superclass with a default `area()` method.
2. `Circle` and `Rectangle` override `area()` with specific implementations.
3. The `shapes` array is of type `Shape[]`, but holds `Circle` and `Rectangle` objects.
4. The enhanced for-loop iterates over the array, calling `area()` on each element.
5. At runtime, the JVM determines the actual type of each object and invokes the appropriate `area()` method.

### Real-World Cases

- **Collections Framework:** A `List` reference can point to an `ArrayList` or `LinkedList`; calling `add()` works uniformly.
- **JDBC:** A `Connection` reference can point to different database-specific implementations.
- **GUI Frameworks:** A `JComponent` array can hold `JButton`, `JLabel`, and `JTextField` objects, all responding to `paint()`.
- **Plugin Systems:** Plugins implement a common interface, and the host application calls methods polymorphically.

---

## 2. Compile-Time Polymorphism

### Definitions

**Core Definition:** Compile-time polymorphism (also called static polymorphism) is polymorphism resolved at compile time, achieved through method overloading.

**Technical Definition:** Compile-time polymorphism occurs when the compiler determines which method to invoke based on the number, types, and order of arguments at the call site. This is achieved through method overloading, where multiple methods share the same name but have different parameter lists. The compiler selects the most specific applicable method during compilation.

**Beginner-Friendly Explanation:** Compile-time polymorphism is like having multiple tools with the same label ("screwdriver") but different sizes. When you ask for a screwdriver, you specify the size, and the right one is chosen before you even start working.

### Purposes

- To provide multiple ways to perform similar operations with different input types
- To improve code readability by using consistent method names
- To enable the compiler to catch type errors early
- To avoid the overhead of runtime method resolution

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Multiple methods with the same name but different parameters
returnType methodName(type1 param1) { ... }
returnType methodName(type2 param1, type3 param2) { ... }
returnType methodName(type4... param1) { ... }
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Same `methodName` | All overloaded methods share the same name. |
| Different parameter lists | Must differ in number, type, or order of parameters. |
| Return type | May differ but does not participate in resolution. |

**Syntax Rules:**

- Overloaded methods must have different signatures.
- The compiler chooses the most specific applicable method.
- Widening conversions are preferred over boxing/unboxing.
- Varargs methods are considered last.

**Constraints and Limitations:**

- Overloading cannot be based solely on return type.
- Overloading is resolved at compile time; runtime types are not considered.
- Ambiguous overloads cause compile-time errors.

### Annotated Code Example

```java
// File: CompileTimePolymorphismDemo.java
public class CompileTimePolymorphismDemo {

    // Overload 1: int
    public static void print(int value) {
        System.out.println("int: " + value);
    }

    // Overload 2: double
    public static void print(double value) {
        System.out.println("double: " + value);
    }

    // Overload 3: String
    public static void print(String value) {
        System.out.println("String: " + value);
    }

    public static void main(String[] args) {
        print(10);        // Resolves to print(int)
        print(10.5);      // Resolves to print(double)
        print("Hello");   // Resolves to print(String)
    }
}
```

**Expected Output:**
```
int: 10
double: 10.5
String: Hello
```

**Step-by-Step Explanation:**

1. Three `print` methods are declared with different parameter types.
2. At each call site, the compiler examines the argument type and selects the matching overload.
3. `print(10)` matches `print(int)` because `10` is an `int`.
4. `print(10.5)` matches `print(double)`.
5. `print("Hello")` matches `print(String)`.
6. All resolution happens at compile time — no runtime lookup is required.

### Real-World Cases

- **`System.out.println`:** Overloaded for `int`, `double`, `String`, `Object`, etc.
- **`String.valueOf`:** Overloaded for different primitive and reference types.
- **`Math.max`:** Overloaded for `int`, `long`, `float`, `double`.
- **`Collections.sort`:** Overloaded for natural ordering and custom comparator.

---

## 3. Runtime Polymorphism

### Definitions

**Core Definition:** Runtime polymorphism (also called dynamic polymorphism) is polymorphism resolved at runtime, achieved through method overriding.

**Technical Definition:** Runtime polymorphism occurs when a subclass overrides a method from its superclass, and the JVM determines at runtime which implementation to invoke based on the actual type of the object. This is implemented through dynamic method dispatch. The reference type determines which methods are accessible at compile time, but the object type determines which implementation runs at runtime.

**Beginner-Friendly Explanation:** Runtime polymorphism is like calling a "play" command on different devices. You don't know which device you have until you press play — then the correct behaviour happens. In Java, the actual object type determines which method runs.

### Purposes

- To allow a single method call to behave differently based on the actual object type
- To enable the "open-closed principle" — new subclasses can be added without modifying existing code
- To support frameworks and libraries that work with user-defined types
- To implement the Strategy pattern and other behavioural patterns

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Superclass
class Superclass {
    public returnType methodName(parameters) { ... }
}

// Subclass
class Subclass extends Superclass {
    @Override
    public returnType methodName(parameters) { ... }
}

// Usage
Superclass ref = new Subclass();  // Upcasting
ref.methodName();                 // Runtime dispatch
```

**Syntax Rules:**

- The overriding method must have the same signature as the overridden method.
- The return type must be the same or covariant.
- The access modifier cannot be more restrictive.
- The `@Override` annotation is recommended for compile-time verification.

**Constraints and Limitations:**

- `final`, `static`, and `private` methods cannot be overridden.
- Fields are not polymorphic; they are resolved at compile time based on the reference type.
- Constructors are not polymorphic.

### Annotated Code Example

```java
// File: RuntimePolymorphismDemo.java
class Payment {
    public void process(double amount) {
        System.out.println("Processing generic payment: $" + amount);
    }
}

class CreditCardPayment extends Payment {
    @Override
    public void process(double amount) {
        System.out.println("Processing credit card payment: $" + amount);
    }
}

class PayPalPayment extends Payment {
    @Override
    public void process(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
    }
}

public class RuntimePolymorphismDemo {
    public static void main(String[] args) {
        // Array of superclass type holding subclass objects
        Payment[] payments = {
            new Payment(),
            new CreditCardPayment(),
            new PayPalPayment()
        };

        // Polymorphic method calls
        for (Payment payment : payments) {
            payment.process(100.0);
        }
    }
}
```

**Expected Output:**
```
Processing generic payment: $100.0
Processing credit card payment: $100.0
Processing PayPal payment: $100.0
```

**Step-by-Step Explanation:**

1. `Payment` is the superclass with a generic `process()` method.
2. `CreditCardPayment` and `PayPalPayment` override `process()` with specific implementations.
3. The `payments` array holds objects of different types but is declared as `Payment[]`.
4. In the loop, `payment.process(100.0)` is called on each element.
5. At runtime, the JVM inspects the actual type of each object and invokes the corresponding `process()` method.
6. This demonstrates dynamic method dispatch — the same call produces different behaviour based on the object type.

### Real-World Cases

- **JDBC Drivers:** Different database drivers implement the same `Connection` interface differently.
- **Servlet Containers:** `HttpServlet.doGet()` is overridden by user servlets.
- **Event Handling:** `ActionListener.actionPerformed()` is implemented differently by different listeners.
- **Design Patterns:** Strategy, Template Method, and Observer patterns all rely on runtime polymorphism.

---

## 4. Method Overriding

### Definitions

**Core Definition:** Method overriding occurs when a subclass provides a new implementation for a method that is already defined in its superclass.

**Technical Definition:** An instance method in a subclass with the same signature (name, plus the number and type of its parameters) and return type as an instance method in the superclass overrides the superclass's method. The ability of a subclass to override a method allows a class to inherit from a superclass whose behavior is "close enough" and then to modify behavior as needed. An overriding method can also return a subtype of the type returned by the overridden method; this subtype is called a covariant return type.

**Beginner-Friendly Explanation:** Overriding is like a child who inherits a recipe from a parent but decides to change some ingredients. The recipe name stays the same, but the child's version is used when you ask the child to cook.

### Purposes

- To allow a subclass to provide a specific implementation of a method already defined by its superclass
- To enable runtime polymorphism (dynamic method dispatch)
- To customize inherited behaviour for specific subclasses
- To implement the "open-closed principle" by extending behaviour without modifying existing code

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Superclass
public returnType methodName(parameters) { ... }

// Subclass
@Override
public returnType methodName(parameters) { ... }
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@Override` | Optional annotation that instructs the compiler to verify the method actually overrides a superclass method. |
| Same signature | Name and parameter types must match exactly. |
| Same or covariant return type | Return type must be the same or a subtype of the superclass method's return type. |
| Access modifier | Cannot be more restrictive than the overridden method's access modifier. |

**Syntax Rules:**

- The overriding method must have the same signature as the overridden method.
- The return type must be the same or a subtype (covariant return).
- The access modifier cannot be more restrictive.
- The overriding method cannot throw broader checked exceptions.
- `final`, `static`, and `private` methods cannot be overridden.

**Constraints and Limitations:**

- Constructors cannot be overridden.
- Static methods are hidden, not overridden.
- Private methods are not inherited and therefore cannot be overridden.

### Annotated Code Example

```java
// File: OverridingDemo.java
class Animal {
    public void makeSound() {
        System.out.println("Some generic animal sound");
    }

    public void move() {
        System.out.println("Animal moves");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof! Woof!");
    }

    @Override
    public void move() {
        System.out.println("Dog runs on four legs");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

public class OverridingDemo {
    public static void main(String[] args) {
        Animal myAnimal = new Animal();
        Animal myDog = new Dog();      // Upcasting
        Animal myCat = new Cat();      // Upcasting

        myAnimal.makeSound();          // Some generic animal sound
        myDog.makeSound();             // Woof! Woof!
        myCat.makeSound();             // Meow!

        myDog.move();                  // Dog runs on four legs
    }
}
```

**Expected Output:**
```
Some generic animal sound
Woof! Woof!
Meow!
Dog runs on four legs
```

**Step-by-Step Explanation:**

1. `Animal` defines `makeSound()` and `move()`.
2. `Dog` overrides both methods with its own implementations.
3. `Cat` overrides only `makeSound()`.
4. `Animal myDog = new Dog()` upcasts a `Dog` object to an `Animal` reference.
5. When `myDog.makeSound()` is called, the JVM uses dynamic method lookup to determine the actual object type (`Dog`) and invokes `Dog`'s version of `makeSound()`.
6. Similarly, `myCat.makeSound()` invokes `Cat`'s version.
7. `myDog.move()` invokes `Dog`'s overridden `move()` method.

### Real-World Cases

- **`toString()`:** Overridden in almost every custom class to provide meaningful string representation.
- **`equals()` and `hashCode()`:** Overridden for proper object equality and hash-based collections.
- **`compareTo()`:** Overridden in classes implementing `Comparable`.
- **Template Method Pattern:** Abstract methods are overridden in concrete subclasses to fill in specific behaviour.

---

## 5. Dynamic Dispatch

### Definitions

**Core Definition:** Dynamic dispatch (also called dynamic method dispatch or virtual method invocation) is the mechanism by which the JVM determines at runtime which method implementation to invoke based on the actual type of the object.

**Technical Definition:** When an instance method is invoked, the JVM examines the actual class of the object (not the declared type of the reference) and searches for the method implementation starting from that class and moving up the inheritance hierarchy. The first matching implementation found is invoked. This process is known as dynamic method dispatch or virtual method invocation.

**Beginner-Friendly Explanation:** Dynamic dispatch is like asking a question to a group of people. Depending on who is actually in the room (the object type), a different person answers (the method implementation).

### Purposes

- To enable runtime polymorphism by selecting the correct method implementation
- To allow subclasses to override behaviour without modifying the caller's code
- To support the "open-closed principle" and extensibility
- To implement the Strategy and Template Method patterns

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// The JVM performs dynamic dispatch automatically
SuperType ref = new SubType();
ref.methodName();  // JVM looks up the method in SubType first
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `SuperType ref` | The compile-time type of the reference. |
| `new SubType()` | The runtime type of the object. |
| `ref.methodName()` | The JVM searches for `methodName` starting from `SubType`. |

**Syntax Rules:**

- Dynamic dispatch applies only to instance methods (not static, private, or final methods).
- The method must be overridden in the subclass for dynamic dispatch to select a different implementation.
- The JVM maintains a virtual method table (vtable) for each class to speed up method lookup.

**Constraints and Limitations:**

- Fields are not subject to dynamic dispatch.
- Static methods are resolved at compile time.
- Private and final methods are resolved at compile time.

### Annotated Code Example

```java
// File: DynamicDispatchDemo.java
class Vehicle {
    public void start() {
        System.out.println("Vehicle starting");
    }
}

class Car extends Vehicle {
    @Override
    public void start() {
        System.out.println("Car engine starting");
    }
}

class Motorcycle extends Vehicle {
    @Override
    public void start() {
        System.out.println("Motorcycle engine starting");
    }
}

public class DynamicDispatchDemo {
    public static void main(String[] args) {
        Vehicle v;

        v = new Car();
        v.start();  // JVM dispatches to Car.start()

        v = new Motorcycle();
        v.start();  // JVM dispatches to Motorcycle.start()
    }
}
```

**Expected Output:**
```
Car engine starting
Motorcycle engine starting
```

**Step-by-Step Explanation:**

1. `Vehicle` is the superclass with a `start()` method.
2. `Car` and `Motorcycle` override `start()`.
3. `v` is declared as `Vehicle`, but its actual object changes.
4. When `v = new Car()`, `v.start()` dispatches to `Car.start()`.
5. When `v = new Motorcycle()`, `v.start()` dispatches to `Motorcycle.start()`.
6. The JVM uses the actual object type (`Car` or `Motorcycle`) to determine which method to invoke.

### Real-World Cases

- **JDBC:** The `DriverManager.getConnection()` method returns a `Connection` object that dispatches to the appropriate database driver.
- **Servlet Containers:** The container calls `service()` on a servlet, which dispatches to the correct HTTP method handler.
- **Collections:** `List.add()` dispatches to `ArrayList.add()` or `LinkedList.add()` depending on the actual object.

---

## 6. Virtual Methods

### Definitions

**Core Definition:** A virtual method is an instance method that can be overridden by subclasses and is subject to dynamic dispatch.

**Technical Definition:** In Java, all non-static, non-final, non-private instance methods are virtual methods by default. The JVM uses a virtual method table (vtable) to resolve method calls at runtime. When a virtual method is invoked, the JVM looks up the method in the vtable of the object's actual class.

**Beginner-Friendly Explanation:** Virtual methods are methods that can be "replaced" by subclasses. In Java, almost all methods are virtual — you can override them. The only exceptions are methods marked `final`, `static`, or `private`.

### Purposes

- To allow subclasses to override behaviour
- To enable runtime polymorphism
- To provide a mechanism for dynamic dispatch

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// By default, all instance methods are virtual
public returnType methodName(parameters) { ... }

// To make a method non-virtual:
public final returnType methodName(parameters) { ... }  // Cannot be overridden
public static returnType methodName(parameters) { ... }  // Not virtual
private returnType methodName(parameters) { ... }        // Not inherited
```

**Component Breakdown:**

| Method Type | Virtual? |
|-------------|----------|
| Instance method (default) | Yes |
| `final` method | No |
| `static` method | No |
| `private` method | No |

**Syntax Rules:**

- All instance methods are virtual unless marked `final`.
- Static methods are not virtual; they are resolved at compile time.
- Private methods are not virtual; they are not inherited.

**Constraints and Limitations:**

- Overusing `final` can prevent flexibility.
- Virtual method calls have a slight performance overhead compared to non-virtual calls (though JIT optimizations often mitigate this).

### Annotated Code Example

```java
// File: VirtualMethodDemo.java
class Base {
    // Virtual method (can be overridden)
    public void virtualMethod() {
        System.out.println("Base virtual method");
    }

    // Non-virtual method (final)
    public final void nonVirtualMethod() {
        System.out.println("Base non-virtual method");
    }
}

class Derived extends Base {
    @Override
    public void virtualMethod() {
        System.out.println("Derived virtual method");
    }

    // Cannot override nonVirtualMethod() — it's final
}

public class VirtualMethodDemo {
    public static void main(String[] args) {
        Base obj = new Derived();
        obj.virtualMethod();      // Derived virtual method (dynamic dispatch)
        obj.nonVirtualMethod();   // Base non-virtual method (compile-time resolution)
    }
}
```

**Expected Output:**
```
Derived virtual method
Base non-virtual method
```

**Step-by-Step Explanation:**

1. `virtualMethod()` is virtual and can be overridden; `nonVirtualMethod()` is final and cannot be overridden.
2. `Base obj = new Derived()` upcasts a `Derived` object.
3. `obj.virtualMethod()` dispatches to `Derived.virtualMethod()` at runtime.
4. `obj.nonVirtualMethod()` is resolved at compile time and always calls `Base.nonVirtualMethod()`.

### Real-World Cases

- **`Object.toString()`:** A virtual method overridden by most classes.
- **`Object.equals()`:** A virtual method overridden for value equality.
- **`InputStream.read()`:** A virtual method overridden by `FileInputStream`, `ByteArrayInputStream`, etc.

---

## 7. Upcasting

### Definitions

**Core Definition:** Upcasting is the act of treating a subclass object as an object of its superclass type.

**Technical Definition:** Upcasting involves assigning a reference of a subclass type to a reference variable of a superclass type. This is always safe and implicit in Java because a subclass object "is-a" superclass object. Upcasting is the foundation of polymorphism.

**Beginner-Friendly Explanation:** Upcasting is like saying "a dog is an animal." You can always treat a specific type (dog) as a more general type (animal) without any risk.

### Purposes

- To enable polymorphic behaviour by using a general reference type
- To allow objects of different subclasses to be handled uniformly
- To provide flexibility in method parameters and collections

### Syntax Rules and Structure

```java
SuperType ref = new SubType();  // Implicit upcasting
```

**Syntax Rules:**

- Upcasting is implicit and does not require a cast.
- The subclass object can only access members defined in the superclass type (at compile time).
- The actual object type is preserved for method dispatch.

**Constraints and Limitations:**

- After upcasting, only methods declared in the superclass are accessible without downcasting.
- Fields are accessed based on the reference type, not the object type.

### Annotated Code Example

```java
// File: UpcastingDemo.java
class Animal {
    public void eat() {
        System.out.println("Animal eating");
    }
}

class Dog extends Animal {
    public void bark() {
        System.out.println("Dog barking");
    }
}

public class UpcastingDemo {
    public static void main(String[] args) {
        // Upcasting: Dog object assigned to Animal reference
        Animal animal = new Dog();

        animal.eat();   // Inherited method — accessible

        // animal.bark();  // COMPILE ERROR: bark() not defined in Animal

        // To call bark(), downcast back to Dog
        if (animal instanceof Dog) {
            ((Dog) animal).bark();
        }
    }
}
```

**Expected Output:**
```
Animal eating
Dog barking
```

**Step-by-Step Explanation:**

1. `Dog` extends `Animal` and adds `bark()`.
2. `Animal animal = new Dog()` upcasts the `Dog` object.
3. `animal.eat()` is accessible because `eat()` is defined in `Animal`.
4. `animal.bark()` would cause a compile error because `bark()` is not in `Animal`.
5. Downcasting with `(Dog) animal` restores access to `bark()`.

### Real-World Cases

- **Collections:** `List<String> list = new ArrayList<>()` — `ArrayList` upcast to `List`.
- **Method Parameters:** `void process(Animal a)` can accept `Dog`, `Cat`, etc.
- **Event Handling:** `ActionEvent` is upcast to `EventObject` in some APIs.

---

## 8. Downcasting

### Definitions

**Core Definition:** Downcasting is the act of treating a superclass reference as a reference to a subclass type.

**Technical Definition:** Downcasting involves casting a superclass reference back to a subclass type. Unlike upcasting, downcasting requires an explicit cast and can throw a `ClassCastException` if the object is not actually an instance of the target type.

**Beginner-Friendly Explanation:** Downcasting is like saying "this animal is actually a dog." You can only do this if the animal really is a dog — otherwise, you'll get an error.

### Purposes

- To access subclass-specific methods and fields after upcasting
- To recover the specific type of an object when needed
- To implement certain design patterns (e.g., Visitor)

### Syntax Rules and Structure

```java
SubType ref = (SubType) superTypeRef;  // Explicit downcast
```

**Syntax Rules:**

- Downcasting requires an explicit cast.
- It throws `ClassCastException` if the object is not an instance of the target type.
- Always use `instanceof` to check before downcasting.

**Constraints and Limitations:**

- Downcasting can fail at runtime if the object is not of the expected type.
- Excessive downcasting is a code smell; consider redesigning with polymorphism.

### Annotated Code Example

```java
// File: DowncastingDemo.java
class Animal {
    public void eat() {
        System.out.println("Animal eating");
    }
}

class Dog extends Animal {
    public void bark() {
        System.out.println("Dog barking");
    }
}

public class DowncastingDemo {
    public static void main(String[] args) {
        Animal animal = new Dog();  // Upcasting

        // Safe downcasting with instanceof check
        if (animal instanceof Dog) {
            Dog dog = (Dog) animal;
            dog.bark();  // Dog barking
        }

        // Unsafe downcasting — throws ClassCastException
        Animal anotherAnimal = new Animal();
        try {
            Dog dog = (Dog) anotherAnimal;  // ClassCastException
        } catch (ClassCastException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Dog barking
Caught: class Animal cannot be cast to class Dog
```

**Step-by-Step Explanation:**

1. `Animal animal = new Dog()` upcasts a `Dog` object.
2. `animal instanceof Dog` checks the actual type before downcasting.
3. `(Dog) animal` downcasts and allows access to `bark()`.
4. Attempting to downcast an `Animal` object to `Dog` throws `ClassCastException` because the object is not actually a `Dog`.

### Real-World Cases

- **Legacy Code:** Downcasting is sometimes necessary when working with older APIs.
- **Object Serialization:** Reading objects from a stream may require downcasting.
- **Generic Collections:** Before Java 5 generics, downcasting was common when retrieving elements from collections.

---

## 9. instanceof

### Definitions

**Core Definition:** The `instanceof` operator is a binary operator that checks whether an object is an instance of a particular type.

**Technical Definition:** The `instanceof` operator returns `true` if the object on the left is an instance of the class or interface on the right, or a subclass or implementing class thereof. It is commonly used before downcasting to avoid `ClassCastException`. As of Java 16, pattern matching for `instanceof` allows the type check and cast to be combined.

**Beginner-Friendly Explanation:** `instanceof` is like asking "Is this a...?" — "Is this animal a dog?" If yes, you can safely treat it as a dog.

### Purposes

- To check the actual type of an object before downcasting
- To implement type-specific behaviour in polymorphic code
- To enable pattern matching (Java 16+)

### Syntax Rules and Structure

**Traditional Syntax:**

```java
if (object instanceof Type) {
    Type ref = (Type) object;
    // use ref
}
```

**Pattern Matching Syntax (Java 16+):**

```java
if (object instanceof Type ref) {
    // use ref directly
}
```

**Syntax Rules:**

- `instanceof` returns `false` if the object is `null`.
- It checks the runtime type of the object, not the reference type.
- Pattern matching eliminates the need for an explicit cast.

**Constraints and Limitations:**

- `instanceof` cannot be used with primitive types.
- Excessive use of `instanceof` may indicate a design problem.

### Annotated Code Example

```java
// File: InstanceofDemo.java
class Animal {
    public void eat() {
        System.out.println("Animal eating");
    }
}

class Dog extends Animal {
    public void bark() {
        System.out.println("Dog barking");
    }
}

public class InstanceofDemo {
    public static void main(String[] args) {
        Animal animal = new Dog();

        // Traditional instanceof
        if (animal instanceof Dog) {
            Dog dog = (Dog) animal;
            dog.bark();  // Dog barking
        }

        // Pattern matching (Java 16+)
        if (animal instanceof Dog dog) {
            dog.bark();  // Dog barking
        }

        // instanceof with null
        Animal nullAnimal = null;
        System.out.println(nullAnimal instanceof Animal);  // false
    }
}
```

**Expected Output:**
```
Dog barking
Dog barking
false
```

**Step-by-Step Explanation:**

1. `animal instanceof Dog` checks if the actual object is a `Dog`.
2. Traditional syntax requires an explicit cast after the check.
3. Pattern matching syntax combines the check and cast.
4. `null instanceof Animal` returns `false` because `null` is not an instance of any type.

### Real-World Cases

- **Event Handling:** Checking the type of an `Event` before processing.
- **Visitor Pattern:** Determining which `accept()` method to call.
- **Serialization:** Checking object types during deserialization.

---

## 10. Pattern Matching

### Definitions

**Core Definition:** Pattern matching for `instanceof` is a Java 16+ feature that allows the type check and cast to be combined into a single expression.

**Technical Definition:** Pattern matching for `instanceof` eliminates the need for explicit casting after a type check. The syntax `object instanceof Type variable` checks if `object` is an instance of `Type` and, if so, binds it to `variable` of type `Type`. The variable is in scope only where the pattern is guaranteed to match.

**Beginner-Friendly Explanation:** Pattern matching is like saying "If this animal is a dog, call it 'dog' and use that name." You don't have to cast separately — it's all in one step.

### Purposes

- To simplify code by eliminating redundant casts
- To improve readability and reduce boilerplate
- To make type checks more concise and less error-prone

### Syntax Rules and Structure

```java
if (object instanceof Type variable) {
    // variable is in scope here
}

// With negation (Java 17+):
if (!(object instanceof Type variable)) {
    // variable is not in scope here
}
```

**Syntax Rules:**

- The variable is in scope only where the pattern is guaranteed to match.
- Pattern matching works with `instanceof` and `switch` (Java 21+).
- The variable cannot be reassigned within the pattern block.

**Constraints and Limitations:**

- Pattern matching for `instanceof` was previewed in Java 14, finalized in Java 16.
- Pattern matching for `switch` was finalized in Java 21.

### Annotated Code Example

```java
// File: PatternMatchingDemo.java
class Animal {
    public void eat() {
        System.out.println("Animal eating");
    }
}

class Dog extends Animal {
    public void bark() {
        System.out.println("Dog barking");
    }
}

class Cat extends Animal {
    public void meow() {
        System.out.println("Cat meowing");
    }
}

public class PatternMatchingDemo {
    public static void describe(Animal animal) {
        // Pattern matching for instanceof
        if (animal instanceof Dog dog) {
            dog.bark();
        } else if (animal instanceof Cat cat) {
            cat.meow();
        } else {
            System.out.println("Unknown animal");
        }
    }

    public static void main(String[] args) {
        describe(new Dog());  // Dog barking
        describe(new Cat());  // Cat meowing
        describe(new Animal());  // Unknown animal
    }
}
```

**Expected Output:**
```
Dog barking
Cat meowing
Unknown animal
```

**Step-by-Step Explanation:**

1. `describe` takes an `Animal` parameter.
2. `animal instanceof Dog dog` checks if `animal` is a `Dog` and binds it to `dog`.
3. If true, `dog.bark()` is called without an explicit cast.
4. Similarly for `Cat`.
5. The final `else` handles the case where the animal is neither a `Dog` nor a `Cat`.

### Real-World Cases

- **Visitor Pattern:** Pattern matching simplifies type-specific processing.
- **Event Handling:** Dispatching events to different handlers based on type.
- **Parsing:** Processing different token types in a parser.

---

## 11. Polymorphism Best Practices

### Definitions

**Core Definition:** Polymorphism best practices are guidelines for using polymorphism effectively and avoiding common pitfalls.

**Technical Definition:** Best practices derive from *Effective Java* and object-oriented design principles. Key practices include programming to interfaces, using abstract classes for shared code, favouring composition over inheritance, and avoiding downcasting when possible.

**Beginner-Friendly Explanation:** Polymorphism best practices are like the rules for using a universal remote — program to the interface, not the implementation, and don't force devices to do things they weren't designed for.

### Purposes

- To write flexible, maintainable, and extensible code
- To reduce coupling between components
- To avoid runtime errors from unsafe downcasting
- To leverage the full power of object-oriented design

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Program to interfaces | Use `List` instead of `ArrayList` as variable types. |
| Use abstract classes for shared code | Put common implementation in abstract base classes. |
| Favour composition over inheritance | Use "has-a" instead of "is-a" when possible. |
| Avoid downcasting | If you need to downcast, reconsider the design. |
| Use `@Override` | Catch errors at compile time. |
| Document the contract | Specify behaviour that subclasses must follow. |
| Keep hierarchies shallow | Deep hierarchies are hard to understand. |
| Use the Liskov Substitution Principle | Subtypes must be substitutable for their base types. |

**Syntax Rules:**

- Declare variables using the most general type that supports the required operations.
- Use interfaces for defining contracts.
- Use abstract classes for partial implementations.

**Constraints and Limitations:**

- Overusing polymorphism can lead to complex designs.
- Programming to interfaces adds a level of indirection.

### Annotated Code Example

```java
// File: PolymorphismBestPractices.java
import java.util.ArrayList;
import java.util.List;

// GOOD: Program to the interface
public class PolymorphismBestPractices {

    // Method takes List (interface) not ArrayList (implementation)
    public static double sumAll(List<? extends Number> numbers) {
        double total = 0;
        for (Number n : numbers) {
            total += n.doubleValue();
        }
        return total;
    }

    public static void main(String[] args) {
        // GOOD: Use interface type for variable
        List<Integer> intList = new ArrayList<>();
        intList.add(1);
        intList.add(2);
        intList.add(3);

        System.out.println("Sum: " + sumAll(intList));  // Sum: 6.0

        // BAD: Using concrete type (less flexible)
        // ArrayList<Integer> badList = new ArrayList<>();
    }
}
```

**Expected Output:**
```
Sum: 6.0
```

**Step-by-Step Explanation:**

1. `sumAll` takes `List<? extends Number>`, making it flexible for any list of numbers.
2. `intList` is declared as `List<Integer>`, not `ArrayList<Integer>`.
3. This allows switching to a different `List` implementation without changing the code.
4. The method works with `Integer`, `Double`, `Float`, etc.

### Real-World Cases

- **Spring Framework:** Uses interfaces extensively for dependency injection.
- **JDBC:** `Connection`, `Statement`, `ResultSet` are all interfaces.
- **Collections Framework:** `List`, `Set`, `Map` are interfaces; `ArrayList`, `HashSet`, `HashMap` are implementations.

---

## References Links

- Polymorphism — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html
- Java Language Specification, §8.4.8: Inheritance, Overriding, and Hiding - https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html
- Java Language Specification, §15.12.4: Run-Time Evaluation of Method Invocation - https://docs.oracle.com/javase/specs/jls/se23/jls23.pdf
- The Java Virtual Machine Specification: Dynamic Dispatch - https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html
- Overriding and Hiding Methods — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/override.html
- Using the Keyword super — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/IandI/super.html
- Pattern Matching for instanceof — Oracle Java Documentation - https://docs.oracle.com/en/java/javase/16/language/pattern-matching-instanceof.html
- Effective Java (3rd Edition) by Joshua Bloch — Items 64: Refer to objects by their interfaces
- SEI CERT Oracle Coding Standard — MET33-J: Do not subject overloaded methods to polymorphic invocations - https://wiki.sei.cmu.edu/confluence/display/java/MET33-J.+Do+not+subject+overloaded+methods+to+polymorphic+invocations
- Static vs Dynamic Binding in Java — GeeksforGeeks - https://www.geeksforgeeks.org/static-vs-dynamic-binding-in-java/
- Java Polymorphism — Baeldung - https://www.baeldung.com/java-polymorphism