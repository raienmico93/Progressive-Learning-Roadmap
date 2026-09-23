# Encapsulation: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Encapsulation** is the object-oriented programming principle of bundling data (fields) and the methods that operate on that data into a single unit (a class), while hiding the internal state of an object from external, uncontrolled access.

### Technical Definition

Encapsulation is a principle of object-oriented programming in which implementation details of a class are hidden from its client, helping to separate the concerns of the client and the implementer and maintain class invariants . It is achieved through **access modifiers** (primarily `private` for fields) and **public methods** (accessors and mutators) that provide controlled access to the internal state. The object itself governs all internal state-changes, and methods can ensure only valid state changes occur .

### Beginner-Friendly Explanation

Think of a class as a vending machine. The internal mechanism (fields) is hidden behind a panel—you can't reach in and grab a snack directly. Instead, you interact through a controlled interface: you insert money and press a button (methods). The machine decides whether the transaction is valid (enough money, valid selection) and only then dispenses the snack. Encapsulation protects the internal state and ensures it can only be modified in valid, controlled ways.

### Key Characteristics

- **Data hiding**: Fields are declared `private`, preventing direct external access .
- **Controlled access**: Public methods (getters and setters) provide the only way to read or modify state.
- **Invariant protection**: Constructors and setters validate inputs to ensure the object's state remains consistent .
- **Implementation hiding**: Clients depend on the interface, not the implementation, allowing internal changes without breaking callers.
- **Defensive copying**: Internal mutable objects are copied before being returned to prevent external corruption .

### Prerequisites

- Basic Java syntax (classes, fields, methods).
- Understanding of access modifiers (`private`, `public`, `protected`, package-private).
- Familiarity with constructors and object creation.

### Related Programming Areas

- **Object-Oriented Design**: Abstraction, inheritance, and polymorphism.
- **API Design**: Public interfaces vs. private implementation.
- **Security**: Preventing state corruption and information leakage .
- **Immutability**: Records and immutable classes as a stronger form of encapsulation.

### Core Concepts / Features

1. Private Fields and Public Methods
2. Getters and Setters (Accessors and Mutators)
3. Controlled State Modification and Data Hiding
4. Invariant Protection (Validation in Setters/Constructors)
5. Defensive Copying
6. Modern Java Records as Immutable Data Carriers


## Core Concept 1: Private Fields and Public Methods

### Definitions

**Core Definition**: Private fields are instance variables declared with the `private` access modifier, accessible only within the class itself. Public methods are declared with `public` and form the controlled interface through which external code interacts with the object.

**Technical Definition**: Encapsulation is implemented by declaring fields `private` and providing `public` methods for access. The `private` modifier restricts access to the class in which the field is declared . Public methods expose the intended behavior while hiding the underlying representation. This separation allows the implementer to change the internal representation without affecting clients that depend only on the public interface.

**Beginner-Friendly Explanation**: A private field is like the combination to a safe—only the class that owns the safe knows it. Public methods are like the keypad on the outside: you can open the safe only by entering the correct code through the keypad, not by cracking it open.

### Purposes

- To hide the internal representation of an object from external code.
- To provide a stable public interface that insulates clients from implementation changes.
- To allow the class to enforce its own rules about how state is accessed and modified.
- To support the class invariant by controlling all state changes.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class ClassName {
    private Type fieldName;  // Private field

    public Type getFieldName() {  // Public accessor
        return fieldName;
    }

    public void setFieldName(Type value) {  // Public mutator
        fieldName = value;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `private Type fieldName` | Field accessible only within the class. |
| `public Type getFieldName()` | Public method returning the field value. |
| `public void setFieldName(Type value)` | Public method modifying the field value. |

**Syntax Rules:**

- Fields are typically declared `private` to enforce encapsulation .
- Public methods provide the only external access to private fields.
- The class can omit getters or setters for fields that should not be accessed externally.
- Access modifiers can be applied to classes, methods, and fields .

**Constraints and Limitations:**

- Private fields cannot be accessed directly from outside the class, causing compile-time errors.
- Not all classes need accessors and mutators; some classes may have no public setters .
- Over-exposing fields via getters and setters can still break encapsulation if the returned objects are mutable.

### Annotated Complete Code Examples

**Example 1: A Point Class with Private Fields and Public Methods**

```java
/**
 * A 2D point with encapsulated coordinates.
 */
public class Point2d {
    // Private fields: only accessible within this class
    private double xCoord;
    private double yCoord;

    /** Two-argument constructor. **/
    public Point2d(double x, double y) {
        xCoord = x;
        yCoord = y;
    }

    /** Default constructor; initializes to (0, 0). **/
    public Point2d() {
        this(0, 0); // Delegates to the two-argument constructor
    }

    // Accessors (getters)
    public double getX() { return xCoord; }
    public double getY() { return yCoord; }

    // Mutators (setters)
    public void setX(double x) { xCoord = x; }
    public void setY(double y) { yCoord = y; }

    @Override
    public String toString() {
        return "(" + xCoord + ", " + yCoord + ")";
    }

    public static void main(String[] args) {
        Point2d p1 = new Point2d();
        Point2d p2 = new Point2d(3.04, -5.612);

        p1.setX(15.1);
        p1.setY(12.67);

        System.out.println("p1 = " + p1);
        System.out.println("p2 = " + p2);
        System.out.println("p2.getX() = " + p2.getX());
    }
}
```

**Expected Output:**

```
p1 = (15.1, 12.67)
p2 = (3.04, -5.612)
p2.getX() = 3.04
```

**Why This Output Occurs:**
- `xCoord` and `yCoord` are private; they cannot be accessed directly from `main` .
- `setX` and `setY` modify the private fields through public methods.
- `getX` and `getY` read the private fields through public accessors.
- The `toString` method provides a readable representation.

**Step-by-Step Setup Guide:**
1. Create `Point2d.java`.
2. Compile with `javac Point2d.java`.
3. Run with `java Point2d`.
4. Observe the output.

### Real-World Cases

- **Entity classes**: `User`, `Product`, `Order` with private fields and public getters/setters.
- **Configuration objects**: Private fields with controlled access to settings.
- **Framework components**: Spring beans and JPA entities encapsulate their state.

### References

- Caltech CS11 – Abstraction and Encapsulation - http://courses.cms.caltech.edu/cs11/material/java/donnie/lectures/cs11-java-lec1.pdf
- Oracle Java Tutorials – Answers to Questions and Exercises: Object-Oriented Programming Concepts - https://docs.oracle.com/javase/tutorial/java/concepts/QandE/answers.html
- Cornell CS 2110 – Classes and Encapsulation - https://www.cs.cornell.edu/courses/cs2110/2026sp/lectures/lec08/


## Core Concept 2: Getters and Setters (Accessors and Mutators)

### Definitions

**Core Definition**: **Accessors** (getters) are public methods that return the value of a private field. **Mutators** (setters) are public methods that modify the value of a private field.

**Technical Definition**: Accessor methods provide read access to private fields, while mutator methods provide controlled write access. Java naming conventions dictate that accessors typically start with `get` (or `is` for boolean fields), and mutators start with `set` . Not all classes have accessors and mutators; some may have only getters (for immutable state) or neither . Setters can and usually should perform input validation and sanitization before setting internal fields .

**Beginner-Friendly Explanation**: Getters are like asking "What's the current setting?" Setters are like saying "Change the setting to this value." The class can decide whether to allow the change and whether to validate the new value first.

### Purposes

- To provide controlled read and write access to private fields.
- To enforce validation rules before state changes.
- To allow the internal representation to change without affecting clients.
- To support the JavaBeans convention for frameworks and tools.

### Syntax Rules and Structure

**Complete General Syntax (Accessor):**

```java
public Type getFieldName() {
    return fieldName;
}
```

**Complete General Syntax (Mutator):**

```java
public void setFieldName(Type value) {
    // optional validation
    fieldName = value;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `public Type getFieldName()` | Returns the field value. |
| `public void setFieldName(Type value)` | Sets the field value (with optional validation). |
| `isFieldName()` | For boolean fields, convention uses `is` instead of `get` . |

**Syntax Rules:**

- Accessors start with `get` (or `is` for booleans); mutators start with `set` .
- Getters take no parameters and return the field type.
- Setters take one parameter of the field type and return `void`.
- Setters can validate the input before assignment.

**Constraints and Limitations:**

- Not all fields need getters or setters; some can be omitted to hide internal state .
- Getters that return mutable objects can break encapsulation unless defensive copies are made .
- Overuse of setters can lead to objects with unconstrained mutable state.

### Annotated Complete Code Examples

**Example 1: Getters and Setters with Validation**

```java
/**
 * A class demonstrating getters and setters with validation.
 */
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        setName(name);
        setAge(age);
    }

    // Getter for name
    public String getName() {
        return name;
    }

    // Setter for name with validation
    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be null or empty");
        }
        this.name = name.trim();
    }

    // Getter for age
    public int getAge() {
        return age;
    }

    // Setter for age with validation
    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Age must be between 0 and 150");
        }
        this.age = age;
    }

    public static void main(String[] args) {
        Person p = new Person("Alice", 30);
        System.out.println("Name: " + p.getName() + ", Age: " + p.getAge());

        p.setName("Bob");
        p.setAge(25);
        System.out.println("Updated: " + p.getName() + ", " + p.getAge());

        try {
            p.setAge(-5); // Invalid
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Name: Alice, Age: 30
Updated: Bob, 25
Error: Age must be between 0 and 150
```

**Why This Output Occurs:**
- The constructor calls `setName` and `setAge`, which validate the inputs.
- `setAge(-5)` throws `IllegalArgumentException` because the age is out of range.
- The validation ensures the object's state remains valid.

### Real-World Cases

- **JavaBeans**: Frameworks rely on getter/setter conventions for property access.
- **JPA/Hibernate**: Entity fields use getters/setters for persistence mapping.
- **Validation frameworks**: Annotations like `@NotNull` and `@Size` are applied to fields with setters.

### References

- Caltech CS11 – Java Method Naming Conventions - http://courses.cms.caltech.edu/cs11/material/java/donnie/lectures/cs11-java-lec1.pdf
- SEI CERT – OBJ09-J: Defensively copy private mutable class members - https://wiki.sei.cmu.edu/confluence/plugins/viewsource/viewpagesrc.action?pageId=88882084
- Oracle Java Tutorials – Answers to Questions and Exercises - https://docs.oracle.com/javase/tutorial/java/concepts/QandE/answers.html


## Core Concept 3: Controlled State Modification and Data Hiding

### Definitions

**Core Definition**: **Controlled state modification** means that all changes to an object's internal state occur through well-defined methods that enforce the class's rules. **Data hiding** means that the internal representation is not exposed to external code.

**Technical Definition**: Encapsulation allows an object to protect its internal state from external access and modification; the object itself governs all internal state-changes, and methods can ensure only valid state changes . Data hiding is achieved by declaring fields `private` and accessing them only through publicly exposed methods . This separation allows the implementer to change the internal representation without affecting clients and ensures the class invariant holds at the start and end of any non-private method call .

**Beginner-Friendly Explanation**: A car's engine is hidden under the hood. You control it through the steering wheel, pedals, and ignition—not by reaching in and moving parts directly. The car's design ensures you can only make valid changes (you can't shift into reverse while going 60 mph).

### Purposes

- To ensure that all state changes are valid and consistent.
- To hide the internal representation so it can change without breaking clients.
- To maintain the class invariant across all method calls.
- To prevent external code from corrupting internal state.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class ClassName {
    private Type field; // Hidden state

    // Controlled modification through methods
    public void modifyField(Type value) {
        // validation, normalization, or transformation
        field = value;
    }
}
```

**Syntax Rules:**

- All fields are declared `private` to hide them from external access .
- Public methods provide the only means of modifying state.
- Methods can validate, normalize, or reject invalid inputs.
- The class invariant must hold at the start and end of any non-private method call .

**Constraints and Limitations:**

- Hiding fields does not hide mutable objects returned by getters; defensive copying is required .
- Public final fields are sometimes acceptable for constants but not for mutable state.
- The class invariant must be maintained by all methods, including constructors and mutators.

### Annotated Complete Code Examples

**Example 1: Controlled State with Invariant Protection**

```java
/**
 * A bank account with controlled state modification.
 */
public class BankAccount {
    private double balance; // Hidden state

    public BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        this.balance = initialBalance;
    }

    // Controlled deposit
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balance += amount;
    }

    // Controlled withdrawal with invariant check
    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal must be positive");
        }
        if (amount > balance) {
            throw new IllegalStateException("Insufficient funds");
        }
        balance -= amount;
    }

    // Read-only access
    public double getBalance() {
        return balance;
    }

    public static void main(String[] args) {
        BankAccount account = new BankAccount(100.0);
        account.deposit(50.0);
        System.out.println("Balance: " + account.getBalance());

        try {
            account.withdraw(200.0); // More than balance
        } catch (IllegalStateException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Balance: 150.0
Error: Insufficient funds
```

**Why This Output Occurs:**
- `deposit(50.0)` adds to the balance: 100 + 50 = 150.
- `withdraw(200.0)` checks if `amount > balance` and throws `IllegalStateException` because the invariant (balance must not go negative) would be violated.
- The state is protected from invalid modifications.

### Real-World Cases

- **Banking systems**: Account balances modified only through controlled deposit/withdraw operations.
- **Inventory systems**: Stock levels modified only through validated add/remove operations.
- **Game state**: Player health modified only through damage/healing methods with constraints.

### References

- Caltech CS11 – Abstraction and Encapsulation - http://courses.cms.caltech.edu/cs11/material/java/donnie/lectures/cs11-java-lec1.pdf
- Cornell CS 2110 – Class Invariants - https://www.cs.cornell.edu/courses/cs2110/2026sp/lectures/lec08/
- Oracle Java Tutorials – Answers to Questions and Exercises - https://docs.oracle.com/javase/tutorial/java/concepts/QandE/answers.html


## Core Concept 4: Invariant Protection (Validation in Setters/Constructors)

### Definitions

**Core Definition**: **Invariant protection** is the practice of validating inputs in constructors and setters to ensure that an object's state remains consistent and valid throughout its lifetime.

**Technical Definition**: A class invariant includes all properties that must be enforced on the fields. It is the responsibility of the implementer to ensure that the class invariant holds at the start and end of any non-private instance method call . Constructors must fully initialize instances so that the invariant is true from the moment of creation . Setters perform input validation and sanitization before setting internal fields .

**Beginner-Friendly Explanation**: An invariant is a rule that must always be true—like "a bank balance can never be negative." Constructors and setters are the gatekeepers: they check inputs and reject anything that would break the rule. If you try to withdraw more than your balance, the setter (or withdrawal method) says "No."

### Purposes

- To ensure objects are always in a valid, consistent state.
- To catch invalid inputs early, before they corrupt the object's state.
- To make the class's rules explicit and enforceable.
- To prevent subtle bugs caused by invalid state.

### Syntax Rules and Structure

**Complete General Syntax (Constructor Validation):**

```java
public ClassName(Type param) {
    if (!isValid(param)) {
        throw new IllegalArgumentException("Invalid parameter: " + param);
    }
    this.field = param;
}
```

**Complete General Syntax (Setter Validation):**

```java
public void setField(Type value) {
    if (!isValid(value)) {
        throw new IllegalArgumentException("Invalid value: " + value);
    }
    this.field = value;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `isValid(param)` | A check that the input satisfies the invariant. |
| `throw new IllegalArgumentException(...)` | Signals that the input is invalid. |
| `this.field = param` | Assigns the validated value. |

**Syntax Rules:**

- Constructors must initialize all fields so the invariant holds immediately .
- Setters must validate before assignment.
- The invariant must hold at the start and end of every non-private method call .
- Invalid inputs should result in exceptions (e.g., `IllegalArgumentException`).

**Constraints and Limitations:**

- Validation adds overhead; avoid redundant checks in internal methods.
- The invariant must be documented for clients to understand the contract.
- Some invariants are complex and require private helper methods to check.

### Annotated Complete Code Examples

**Example 1: Constructor and Setter Validation**

```java
/**
 * A temperature class that enforces a valid range.
 */
public class Temperature {
    private double celsius;
    private static final double MIN_CELSIUS = -273.15; // Absolute zero

    public Temperature(double celsius) {
        if (celsius < MIN_CELSIUS) {
            throw new IllegalArgumentException(
                "Temperature cannot be below absolute zero: " + MIN_CELSIUS);
        }
        this.celsius = celsius;
    }

    public double getCelsius() {
        return celsius;
    }

    public void setCelsius(double celsius) {
        if (celsius < MIN_CELSIUS) {
            throw new IllegalArgumentException(
                "Temperature cannot be below absolute zero: " + MIN_CELSIUS);
        }
        this.celsius = celsius;
    }

    public static void main(String[] args) {
        Temperature temp = new Temperature(25.0);
        System.out.println("Temperature: " + temp.getCelsius() + "°C");

        temp.setCelsius(30.0);
        System.out.println("Updated: " + temp.getCelsius() + "°C");

        try {
            temp.setCelsius(-300.0); // Below absolute zero
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Temperature: 25.0°C
Updated: 30.0°C
Error: Temperature cannot be below absolute zero: -273.15
```

**Why This Output Occurs:**
- The constructor validates that the initial temperature is at or above absolute zero.
- `setCelsius(-300.0)` throws an exception because it violates the invariant.
- The state remains valid at all times.

### Real-World Cases

- **Financial systems**: Transactions validated against account balance and limits.
- **Booking systems**: Dates validated against availability and constraints.
- **Configuration objects**: Settings validated against allowed ranges.

### References

- Cornell CS 2110 – Class Invariants and Encapsulation - https://www.cs.cornell.edu/courses/cs2110/2026sp/lectures/lec08/
- SEI CERT – OBJ09-J: Defensively copy private mutable class members - https://wiki.sei.cmu.edu/confluence/plugins/viewsource/viewpagesrc.action?pageId=88882084
- Cornell CS 2110 – Constructors and Invariants - https://www.cs.cornell.edu/courses/cs2110/2017fa/L03-Fields/cs2110Fields-6up.pdf


## Core Concept 5: Defensive Copying

### Definitions

**Core Definition**: **Defensive copying** is the practice of creating copies of mutable objects when they are passed into a class (via constructors or setters) or returned from it (via getters), to prevent external code from modifying the internal state.

**Technical Definition**: Returning references to internal mutable members of a class can compromise an application's security, both by breaking encapsulation and by providing the opportunity to corrupt the internal state of the class . Performing a defensive copy before returning a reference to a mutable internal state ensures that the caller can only modify the copy and not the original internal state . Defensive copies are also required when accepting mutable objects as constructor or setter arguments, to prevent external modification of the internal state after construction.

**Beginner-Friendly Explanation**: Imagine you have a diary with a secret entry. If someone asks to read it, you don't hand over the original—you give them a photocopy. If they scribble on the copy, your original is safe. Defensive copying is the same: when returning a mutable object, return a copy; when accepting one, store a copy.

### Purposes

- To protect internal mutable state from external corruption.
- To maintain encapsulation even when mutable objects are involved.
- To prevent security vulnerabilities from exposing sensitive data .
- To ensure that the object's state remains under the class's control.

### Syntax Rules and Structure

**Complete General Syntax (Returning a Copy):**

```java
public Type getField() {
    return new Type(field); // or field.clone()
}
```

**Complete General Syntax (Accepting a Copy):**

```java
public void setField(Type value) {
    this.field = new Type(value); // or value.clone()
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `new Type(field)` | Creates a copy of the mutable object. |
| `field.clone()` | Alternative copy mechanism (if supported). |
| `this.field = new Type(value)` | Stores a copy of the input. |

**Syntax Rules:**

- Return a copy, not the original reference, from getters of mutable fields .
- Store a copy, not the original reference, from constructors and setters of mutable fields.
- For arrays, use `Arrays.copyOf()` or a deep copy if elements are mutable.
- For collections, use `List.copyOf()`, `Set.copyOf()`, etc., for immutable copies.

**Constraints and Limitations:**

- Shallow copies are insufficient if the object contains mutable elements .
- Deep copies are more expensive but necessary for full protection.
- `clone()` can be dangerous with untrusted subclasses .

### Annotated Complete Code Examples

**Example 1: Defensive Copying of a Mutable Field**

```java
import java.util.*;

/**
 * Demonstrates defensive copying to protect internal state.
 */
public class DefensiveCopyDemo {
    private final List<String> items;

    // Constructor: defensive copy of the input list
    public DefensiveCopyDemo(List<String> items) {
        this.items = new ArrayList<>(items); // Copy input
    }

    // Getter: defensive copy of the internal list
    public List<String> getItems() {
        return new ArrayList<>(items); // Copy output
    }

    // Mutator: defensive copy of the input
    public void addItem(String item) {
        items.add(item);
    }

    public static void main(String[] args) {
        List<String> original = new ArrayList<>();
        original.add("a");
        original.add("b");

        DefensiveCopyDemo demo = new DefensiveCopyDemo(original);

        // Modifying original does NOT affect demo
        original.add("c");
        System.out.println("Original: " + original);
        System.out.println("Demo: " + demo.getItems());

        // Modifying the returned list does NOT affect demo
        List<String> returned = demo.getItems();
        returned.add("d");
        System.out.println("Returned after modification: " + returned);
        System.out.println("Demo after returned modification: " + demo.getItems());

        // Adding through the controlled mutator DOES affect demo
        demo.addItem("e");
        System.out.println("Demo after addItem: " + demo.getItems());
    }
}
```

**Expected Output:**

```
Original: [a, b, c]
Demo: [a, b]
Returned after modification: [a, b, d]
Demo after returned modification: [a, b]
Demo after addItem: [a, b, e]
```

**Why This Output Occurs:**
- The constructor copies the input list, so modifying `original` does not affect `demo`.
- `getItems()` returns a copy, so modifying the returned list does not affect `demo`.
- `addItem("e")` modifies the internal list through the controlled mutator.

### Real-World Cases

- **Security-sensitive classes**: Returning `Date` objects without copying exposed a vulnerability in JDK 1.7 .
- **Collections**: `Collections.unmodifiableList()` wraps a list but does not copy; defensive copying is still needed.
- **Configuration objects**: Returning copies of configuration maps to prevent external modification.

### References

- SEI CERT – OBJ09-J: Defensively copy private mutable class members - https://wiki.sei.cmu.edu/confluence/plugins/viewsource/viewpagesrc.action?pageId=88882084
- SEI CERT – OBJ09-J (published version) - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88882088
- CMU Principles of Software Construction – Immutability and Defensive Copies - http://www.cs.cmu.edu/~charlie/courses/17-214/2019-fall/slides/20191001-invariants-immutabilility-and-testing.pdf


## Core Concept 6: Modern Java Records as Immutable Data Carriers

### Definitions

**Core Definition**: A **record** is a special kind of class in Java that acts as a shallowly immutable, transparent carrier for a fixed set of values, called record components .

**Technical Definition**: A record class is a shallowly immutable, transparent carrier for a fixed set of values, called the record components . It has a canonical constructor, a private final field for each component, and a public accessor method for each component . The compiler generates implementations of `equals`, `hashCode`, and `toString` based on all component fields . Records are **shallowly immutable**: the references to components cannot be reassigned, but if a component is a mutable object, its contents can still change . The primary reasons to provide an explicit canonical constructor or accessor methods are to validate arguments, perform defensive copies on mutable components, or normalize groups of components .

**Beginner-Friendly Explanation**: A record is like a sealed envelope with a fixed set of labeled slots. You can read the slots, but you can't change what's in them or add new slots. However, if a slot contains a box (a mutable object), someone could still open the box and change its contents—that's why records are "shallowly" immutable. To make them fully immutable, you need to copy mutable components in the constructor.

### Purposes

- To concisely declare classes that are transparent carriers for immutable data.
- To reduce boilerplate for data classes (no need to write getters, `equals`, `hashCode`, `toString`).
- To provide a clear, concise syntax for value-based classes.
- To support pattern matching and deconstruction in future Java features.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public record RecordName(Type1 component1, Type2 component2) {
    // optional compact constructor for validation/defensive copying
    public RecordName {
        // validation and normalization
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `record` | Keyword declaring a record class. |
| `RecordName` | The record class name. |
| `(Type1 component1, ...)` | The record components (become private final fields). |
| `public RecordName { ... }` | Compact canonical constructor for validation. |

**Syntax Rules:**

- Record components are declared in the record header.
- The compiler generates a private final field, a public accessor, and a canonical constructor for each component .
- The canonical constructor can be declared explicitly for validation or defensive copying .
- The compact constructor syntax omits the parameter list and field assignments .
- Records cannot extend other classes (they implicitly extend `Record`) and cannot declare instance fields.

**Constraints and Limitations:**

- Records are **shallowly immutable**; mutable components (arrays, collections) can still be modified .
- To enforce deep immutability, defensive copies must be made in the canonical constructor .
- Records cannot declare additional instance fields.
- Records are final and cannot be abstract.

### Annotated Complete Code Examples

**Example 1: A Simple Record**

```java
/**
 * A simple record for a 2D point.
 */
public record Point(int x, int y) {
    public static void main(String[] args) {
        Point p = new Point(3, 4);
        System.out.println("Point: " + p);
        System.out.println("x = " + p.x() + ", y = " + p.y());
        System.out.println("Equals: " + p.equals(new Point(3, 4)));
    }
}
```

**Expected Output:**

```
Point: Point[x=3, y=4]
x = 3, y = 4
Equals: true
```

**Why This Output Occurs:**
- The record automatically generates a constructor, accessors (`x()`, `y()`), `toString`, `equals`, and `hashCode`.
- `p.x()` and `p.y()` are the generated accessor methods.
- `equals` compares component values, so two points with the same coordinates are equal.

**Example 2: Record with Defensive Copying**

```java
import java.util.*;

/**
 * A record with a mutable component and defensive copying.
 */
public record UserRecord(String userName, List<String> mailAddresses) {
    // Compact constructor for defensive copying
    public UserRecord {
        mailAddresses = List.copyOf(mailAddresses); // Defensive copy
    }

    public static void main(String[] args) {
        List<String> emails = new ArrayList<>();
        emails.add("alice@example.com");

        UserRecord user = new UserRecord("Alice", emails);

        // Modifying the original list does NOT affect the record
        emails.add("hacked@example.com");
        System.out.println("Record emails: " + user.mailAddresses());
        System.out.println("Original emails: " + emails);
    }
}
```

**Expected Output:**

```
Record emails: [alice@example.com]
Original emails: [alice@example.com, hacked@example.com]
```

**Why This Output Occurs:**
- The compact constructor uses `List.copyOf(mailAddresses)` to create an immutable copy .
- Modifying the original `emails` list does not affect the record's component.
- The record's `mailAddresses()` accessor returns the immutable copy.

### Real-World Cases

- **DTOs (Data Transfer Objects)**: Records are ideal for carrying data between layers.
- **Value objects**: `Point`, `Money`, `PhoneNumber` as records.
- **API responses**: Records for JSON serialization with Jackson.
- **Configuration**: Immutable configuration records.

### References

- OpenJDK – Class Record (Java 20 draft) - https://cr.openjdk.org/~iris/se/20/spec/draft/java-se-20-draft-spec-34/api/java.base/java/lang/Record.html
- Oracle Help Center – Class Record (Java 25) - https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Record.html
- OpenJDK – Records and Defensive Copying - https://mail.openjdk.org/pipermail/amber-dev/2020-June/006161.html
- OpenJDK – List::copyOf in Compact Constructors - https://mail.openjdk.org/pipermail/amber-dev/2020-April/005910.html
- Hebrew University – Immutability and Records - https://courses.cs.tau.ac.il/software1/2324a_new/lectures/pdf/05_interfaces_detailed_new.pdf


## References

- Caltech CS11 – Abstraction and Encapsulation - http://courses.cms.caltech.edu/cs11/material/java/donnie/lectures/cs11-java-lec1.pdf
- Cornell CS 2110 – Classes and Encapsulation - https://www.cs.cornell.edu/courses/cs2110/2026sp/lectures/lec08/
- Oracle Java Tutorials – Answers to Questions and Exercises: Object-Oriented Programming Concepts - https://docs.oracle.com/javase/tutorial/java/concepts/QandE/answers.html
- SEI CERT – OBJ09-J: Defensively copy private mutable class members before returning their references - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88882088
- SEI CERT – OBJ09-J (source view) - https://wiki.sei.cmu.edu/confluence/plugins/viewsource/viewpagesrc.action?pageId=88882084
- CMU Principles of Software Construction – Invariants, Immutability, and Testing - http://www.cs.cmu.edu/~charlie/courses/17-214/2019-fall/slides/20191001-invariants-immutabilility-and-testing.pdf
- OpenJDK – Class Record (Java 20 draft) - https://cr.openjdk.org/~iris/se/20