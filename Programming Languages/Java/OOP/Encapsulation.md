# Java Encapsulation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Encapsulation is a fundamental principle of object-oriented programming that involves bundling data (fields) and the methods that operate on that data into a single unit (a class), while restricting direct access to some of the object's internal components.

**Technical Definition:** Encapsulation in Java is achieved through the use of access modifiers (`private`, `protected`, `public`, and package-private) that control the visibility of classes, fields, methods, and constructors. Data members are declared `private` and accessed through public wrapper methods (getters and setters), ensuring that the internal state of an object can only be modified in controlled ways.

**Beginner-Friendly Explanation:** Think of encapsulation as a medicine capsule. The medicine (data) is sealed inside a protective shell (the class), and you can only interact with it through specific, controlled means (methods). You cannot reach in and grab the medicine directly — you have to ask the capsule to release it. This protects the medicine from contamination and ensures it is used correctly.

### Key Characteristics

- **Data Hiding:** Internal fields are declared `private`, preventing direct access from outside the class.
- **Controlled Access:** Public getter and setter methods provide controlled access to private fields.
- **Validation:** Setters can validate input before modifying the internal state.
- **Modularity:** Classes become self-contained units that can be developed, tested, and maintained independently.
- **Security:** Encapsulation prevents unauthorized access and modification, reducing the risk of security vulnerabilities.
- **Maintainability:** Internal implementation can change without affecting external code, as long as the public API remains consistent.

### Prerequisites

- Basic understanding of Java classes and objects
- Familiarity with method declaration and invocation
- Knowledge of access modifiers and their scope
- Awareness of the difference between primitive types and reference types

### Related Programming Areas

- **Object-Oriented Programming (OOP):** Encapsulation is one of the four fundamental OOP principles (along with inheritance, polymorphism, and abstraction).
- **API Design:** Public methods and classes form the API of a library or framework; encapsulation ensures only intended members are exposed.
- **Security:** Encapsulation is a key defence against malicious code and unintended state modification.
- **Immutability:** Immutable objects rely heavily on encapsulation to prevent state changes after construction.
- **Design Patterns:** Patterns like Builder, Factory, and Singleton all depend on encapsulation to control object creation and access.

### Core Concepts / Features

---

## 1. Encapsulation

### Definitions

**Core Definition:** Encapsulation is the bundling of data and the methods that operate on that data into a single unit, while restricting direct access to the data.

**Technical Definition:** Encapsulation involves declaring class fields as `private` and providing `public` getter and setter methods to access and modify them. This ensures that the internal state of an object cannot be directly manipulated by external code, preserving class invariants and enabling validation and security checks.

**Beginner-Friendly Explanation:** Encapsulation is like a car's engine. You don't directly manipulate the fuel injection or timing belt — you use the accelerator and brake pedals (methods) that the car provides. The engine's internal workings are hidden, and you interact with it through a controlled interface.

### Purposes

- To hide the internal state of an object from external code
- To protect data integrity by controlling how fields are accessed and modified
- To enable validation and security checks in setter methods
- To allow internal implementation to change without breaking external code
- To promote modularity and maintainability

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class ClassName {
    // Private fields (data hiding)
    private type fieldName;

    // Public getter
    public type getFieldName() {
        return fieldName;
    }

    // Public setter with validation
    public void setFieldName(type value) {
        // validation
        this.fieldName = value;
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `private type fieldName` | The field is hidden from external access. |
| `public type getFieldName()` | Returns the field value. |
| `public void setFieldName(type value)` | Sets the field value after validation. |

**Syntax Rules:**

- Fields should be declared `private` unless there is a compelling reason otherwise.
- Getter methods follow the naming convention `getX()` or `isX()` for booleans.
- Setter methods follow the naming convention `setX(value)`.
- Getters and setters should be `public` if the field is part of the public API.

**Constraints and Limitations:**

- Encapsulation does not prevent reflection from accessing private members.
- Over-encapsulation (excessive getters/setters for every field) can lead to anemic domain models.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Encapsulation**

```java
// File: EncapsulationDemo.java
public class EncapsulationDemo {
    // Private fields — hidden from external access
    private String name;
    private int age;

    // Constructor
    public EncapsulationDemo(String name, int age) {
        this.name = name;
        setAge(age);  // Use setter for validation
    }

    // Getter for name
    public String getName() {
        return name;
    }

    // Setter for name with validation
    public void setName(String name) {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        this.name = name;
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
        EncapsulationDemo person = new EncapsulationDemo("Alice", 30);
        System.out.println(person.getName() + " is " + person.getAge());  // Alice is 30

        person.setName("Bob");
        person.setAge(25);
        System.out.println(person.getName() + " is " + person.getAge());  // Bob is 25

        // Invalid age throws exception
        try {
            person.setAge(-5);
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());  // Caught: Age must be between 0 and 150
        }
    }
}
```

**Expected Output:**
```
Alice is 30
Bob is 25
Caught: Age must be between 0 and 150
```

**Step-by-Step Explanation:**

1. Fields `name` and `age` are declared `private`, so they cannot be accessed directly from outside the class.
2. The constructor uses `setAge()` for validation.
3. `getName()` and `getAge()` provide read access.
4. `setName()` and `setAge()` provide write access with validation.
5. Attempting to set an invalid age throws an `IllegalArgumentException`.

### Real-World Cases

- **Domain Objects:** `Customer`, `Order`, and `Product` classes encapsulate their fields and provide validated accessors.
- **JavaBeans:** The JavaBeans specification requires private fields with public getters and setters.
- **Framework Components:** Spring beans and JPA entities rely on encapsulation for dependency injection and persistence.

---

## 2. Access Modifiers

### Definitions

**Core Definition:** Access modifiers are keywords that control the visibility and accessibility of classes, fields, methods, and constructors.

**Technical Definition:** Java provides four access levels: `private` (same class only), package-private (same package only), `protected` (same package and subclasses), and `public` (everywhere). These modifiers determine which classes can access a particular member.

**Beginner-Friendly Explanation:** Access modifiers are like the locks on different doors in a building. `private` is a personal diary — only you can read it. Package-private is a shared office — only your team can access it. `protected` is a family room — your team and your children (subclasses) can use it. `public` is the lobby — anyone can enter.

### Purposes

- To control the visibility and accessibility of class members
- To enforce encapsulation by restricting direct access to internal data
- To define the public API of a class
- To enable inheritance while protecting sensitive members

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// For classes (top-level)
public class ClassName { ... }        // public: everywhere
class ClassName { ... }                // package-private: same package

// For members
private type field;                    // private: same class only
type field;                            // package-private: same package
protected type field;                  // protected: same package + subclasses
public type field;                     // public: everywhere
```

**Component Breakdown:**

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `private` | Yes | No | No | No |
| (package-private) | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes | No |
| `public` | Yes | Yes | Yes | Yes |

**Syntax Rules:**

- Top-level classes can only be `public` or package-private.
- Member fields and methods can use all four access levels.
- The most restrictive access level should be used that makes sense.

**Constraints and Limitations:**

- Access modifiers cannot be applied to local variables.
- The `protected` modifier has special rules for access from subclasses in different packages.

### Annotated Code Example

```java
// File: AccessModifierDemo.java
package com.example;

public class AccessModifierDemo {
    private int privateField = 1;       // Only this class
    int packageField = 2;               // Same package
    protected int protectedField = 3;   // Same package + subclasses
    public int publicField = 4;         // Everywhere

    private void privateMethod() { System.out.println("private"); }
    void packageMethod() { System.out.println("package"); }
    protected void protectedMethod() { System.out.println("protected"); }
    public void publicMethod() { System.out.println("public"); }

    public static void main(String[] args) {
        AccessModifierDemo obj = new AccessModifierDemo();
        // All members accessible within the same class
        System.out.println(obj.privateField);    // 1
        System.out.println(obj.packageField);    // 2
        System.out.println(obj.protectedField);  // 3
        System.out.println(obj.publicField);     // 4
        obj.privateMethod();                     // private
        obj.packageMethod();                     // package
        obj.protectedMethod();                   // protected
        obj.publicMethod();                      // public
    }
}
```

**Expected Output:**
```
1
2
3
4
private
package
protected
public
```

**Step-by-Step Explanation:**

1. `privateField` and `privateMethod()` are accessible only within `AccessModifierDemo`.
2. `packageField` and `packageMethod()` are accessible within the same package.
3. `protectedField` and `protectedMethod()` are accessible within the package and by subclasses.
4. `publicField` and `publicMethod()` are accessible from anywhere.
5. Within the same class, all members are accessible.

### Real-World Cases

- **`java.lang.Object`:** `public` methods like `toString()` and `equals()` are available everywhere.
- **Utility Classes:** `Math` has a `private` constructor to prevent instantiation.
- **Framework Hooks:** `protected` methods in abstract classes allow subclasses to customize behaviour.
- **Package-Private:** Internal helper classes in a package are not exposed to external code.

---

## 3. private

### Definitions

**Core Definition:** The `private` access modifier restricts access to the same class only.

**Technical Definition:** Members declared `private` are accessible only within the body of the class in which they are declared. This is the most restrictive access level and is core to encapsulation.

**Beginner-Friendly Explanation:** `private` is like a personal diary — only you can read and write in it.

### Purposes

- To hide implementation details from external code
- To prevent direct modification of internal state
- To enforce encapsulation and data integrity

### Syntax Rules and Structure

```java
private type fieldName;
private returnType methodName(parameters) { ... }
private ClassName(parameters) { ... }
```

**Syntax Rules:**

- `private` members are accessible only within the same class.
- Even instances of the same class can access each other's private members.
- Private constructors prevent external instantiation.

**Constraints and Limitations:**

- Private members are not inherited by subclasses.
- Reflection can bypass `private` access, but this is discouraged.

### Annotated Code Example

```java
// File: PrivateDemo.java
public class PrivateDemo {
    private int secret = 42;

    private void revealSecret() {
        System.out.println("Secret: " + secret);
    }

    public void showSecret() {
        revealSecret();  // Accessible within the same class
    }

    public static void main(String[] args) {
        PrivateDemo obj = new PrivateDemo();
        obj.showSecret();  // Secret: 42

        // obj.secret;       // Compile error: private access
        // obj.revealSecret(); // Compile error: private access
    }
}
```

**Expected Output:**
```
Secret: 42
```

**Step-by-Step Explanation:**

1. `secret` is `private`, so it cannot be accessed directly from `main`.
2. `revealSecret()` is `private`, so it cannot be called from `main`.
3. `showSecret()` is `public` and calls `revealSecret()` internally.

### Real-World Cases

- **Helper Methods:** Internal utility methods are often `private`.
- **Constants:** `private static final` constants used only within a class.
- **Singleton Pattern:** Private constructors prevent external instantiation.

---

## 4. default (package-private)

### Definitions

**Core Definition:** Package-private (default) access is the access level when no modifier is specified; members are accessible only within the same package.

**Technical Definition:** Members with no access modifier are accessible to any class within the same package, but not from classes in other packages.

**Beginner-Friendly Explanation:** Package-private is like a shared office — only people in the same department (package) can access it.

### Purposes

- To allow collaboration between closely related classes within a package
- To hide members from external packages while sharing them within the package

### Syntax Rules and Structure

```java
type fieldName;                    // No modifier
returnType methodName() { ... }    // No modifier
class ClassName { ... }            // No modifier (top-level)
```

**Syntax Rules:**

- Package-private members are accessible to all classes in the same package.
- They are not accessible from subclasses in different packages.

**Constraints and Limitations:**

- Package-private classes cannot be imported from other packages.
- Changing the package structure can break package-private access.

### Annotated Code Example

```java
// File: PackagePrivateDemo.java
package com.example.internal;

class PackagePrivateDemo {  // package-private class
    int packageField = 10;  // package-private field

    void packageMethod() {  // package-private method
        System.out.println("Package-private method called");
    }
}

// In the same package
package com.example.internal;

public class SamePackageUser {
    public static void main(String[] args) {
        PackagePrivateDemo obj = new PackagePrivateDemo();
        System.out.println(obj.packageField);  // 10
        obj.packageMethod();                    // Package-private method called
    }
}
```

**Expected Output:**
```
10
Package-private method called
```

**Step-by-Step Explanation:**

1. `PackagePrivateDemo` and its members have no access modifier, making them package-private.
2. `SamePackageUser` is in the same package, so it can access them.

### Real-World Cases

- **Internal APIs:** Helper classes used only within a package.
- **Testing:** Test classes in the same package can access package-private members.

---

## 5. protected

### Definitions

**Core Definition:** The `protected` access modifier allows access from the same package and from subclasses in any package.

**Technical Definition:** Protected members are accessible within the same package (like package-private) and additionally by subclasses, even if they are in different packages.

**Beginner-Friendly Explanation:** `protected` is like a family room — your immediate family (same package) and your children (subclasses) can access it, but not the general public.

### Purposes

- To allow subclasses to access and extend behaviour
- To provide hooks for customization in inheritance hierarchies
- To share implementation details with subclasses without exposing them to the world

### Syntax Rules and Structure

```java
protected type fieldName;
protected returnType methodName(parameters) { ... }
```

**Syntax Rules:**

- Protected members are accessible within the same package.
- They are also accessible by subclasses in different packages.
- An overriding method cannot reduce visibility from `protected` to `private`.

**Constraints and Limitations:**

- Protected members can be accessed by any class in the same package, which may be unintended.
- Overuse of `protected` can lead to fragile base class problems.

### Annotated Code Example

```java
// File: ProtectedDemo.java
package com.example.base;

public class ProtectedDemo {
    protected String message = "Protected message";

    protected void display() {
        System.out.println(message);
    }
}

// In a different package
package com.example.derived;

import com.example.base.ProtectedDemo;

public class DerivedClass extends ProtectedDemo {
    public void show() {
        display();  // Accessible because it's a subclass
    }

    public static void main(String[] args) {
        DerivedClass obj = new DerivedClass();
        obj.show();  // Protected message
    }
}
```

**Expected Output:**
```
Protected message
```

**Step-by-Step Explanation:**

1. `message` and `display()` are `protected` in `ProtectedDemo`.
2. `DerivedClass` extends `ProtectedDemo` and is in a different package.
3. `DerivedClass` can access the protected members because it is a subclass.

### Real-World Cases

- **Template Method Pattern:** Protected hook methods allow subclasses to customize behaviour.
- **Abstract Classes:** Protected constructors and methods are common in framework design.
- **`Object.clone()`:** This method is `protected`, allowing subclasses to decide whether to support cloning.

---

## 6. public

### Definitions

**Core Definition:** The `public` access modifier allows access from any other class in any package.

**Technical Definition:** Public members form the API of a class and are accessible from anywhere in the program.

**Beginner-Friendly Explanation:** `public` is like a public library — anyone can walk in and use it.

### Purposes

- To expose functionality to other classes and packages
- To define the public API of a class or library
- To enable polymorphism through overriding

### Syntax Rules and Structure

```java
public type fieldName;
public returnType methodName(parameters) { ... }
public class ClassName { ... }
```

**Syntax Rules:**

- Public methods form the public API and should be documented with Javadoc.
- An overriding method cannot reduce visibility from `public` to a more restrictive level.
- Only one public class per source file is allowed.

**Constraints and Limitations:**

- Public methods become part of the API contract; changing them can break clients.
- Overexposing methods increases the surface area for bugs and security issues.

### Annotated Code Example

```java
// File: PublicDemo.java
public class PublicDemo {
    public String name = "Public";

    public void greet() {
        System.out.println("Hello from " + name);
    }

    public static void main(String[] args) {
        PublicDemo obj = new PublicDemo();
        obj.greet();  // Hello from Public
    }
}

// In another package
package com.example.other;
import com.example.PublicDemo;

public class ExternalUser {
    public static void main(String[] args) {
        PublicDemo obj = new PublicDemo();
        System.out.println(obj.name);  // Public
        obj.greet();                    // Hello from Public
    }
}
```

**Expected Output:**
```
Hello from Public
Public
Hello from Public
```

**Step-by-Step Explanation:**

1. `PublicDemo` and its members are `public`.
2. `ExternalUser` in a different package can access them.

### Real-World Cases

- **Java Standard Library:** Every method in `String`, `Math`, and `List` is public.
- **REST APIs:** Controller methods annotated with `@GetMapping` are public.
- **Service Classes:** Service methods exposed to controllers are public.

---

## 7. Getters

### Definitions

**Core Definition:** A getter is a public method that returns the value of a private field.

**Technical Definition:** Getters (also called accessor methods) follow the naming convention `getX()` for non-boolean fields and `isX()` for boolean fields. They provide read access to private fields without allowing direct modification.

**Beginner-Friendly Explanation:** A getter is like a window into a house — you can look inside but you cannot change anything.

### Purposes

- To provide controlled read access to private fields
- To allow internal representation to change without affecting callers
- To support the JavaBeans property convention

### Syntax Rules and Structure

```java
public type getFieldName() {
    return fieldName;
}

public boolean isFieldName() {  // For boolean fields
    return fieldName;
}
```

**Syntax Rules:**

- Getters take no parameters.
- Getters return the field type (or a copy for mutable objects).
- Boolean getters may use `is` instead of `get`.

**Constraints and Limitations:**

- Returning a mutable object directly breaks encapsulation; return a defensive copy.

### Annotated Code Example

```java
// File: GetterDemo.java
public class GetterDemo {
    private String name;
    private boolean active;

    public GetterDemo(String name, boolean active) {
        this.name = name;
        this.active = active;
    }

    public String getName() {
        return name;
    }

    public boolean isActive() {
        return active;
    }

    public static void main(String[] args) {
        GetterDemo obj = new GetterDemo("Alice", true);
        System.out.println(obj.getName());     // Alice
        System.out.println(obj.isActive());    // true
    }
}
```

**Expected Output:**
```
Alice
true
```

**Step-by-Step Explanation:**

1. `getName()` returns the `name` field.
2. `isActive()` returns the `active` boolean field.

### Real-World Cases

- **JavaBeans:** Getters are required for properties.
- **DTOs:** Data Transfer Objects use getters to expose data.
- **Immutable Objects:** Getters return field values without allowing modification.

---

## 8. Setters

### Definitions

**Core Definition:** A setter is a public method that modifies the value of a private field.

**Technical Definition:** Setters (also called mutator methods) follow the naming convention `setX(value)`. They provide write access to private fields and are the ideal place to implement validation.

**Beginner-Friendly Explanation:** A setter is like a receptionist — you can give them information, but they decide whether to pass it on and how.

### Purposes

- To provide controlled write access to private fields
- To validate input before modifying state
- To enforce class invariants

### Syntax Rules and Structure

```java
public void setFieldName(type value) {
    // validation
    this.fieldName = value;
}
```

**Syntax Rules:**

- Setters take one parameter (the new value).
- Setters return `void`.
- Validation should be performed before assignment.

**Constraints and Limitations:**

- Setters can be omitted to create read-only properties.
- Overuse of setters can lead to anemic domain models.

### Annotated Code Example

```java
// File: SetterDemo.java
public class SetterDemo {
    private int age;

    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age");
        }
        this.age = age;
    }

    public int getAge() {
        return age;
    }

    public static void main(String[] args) {
        SetterDemo obj = new SetterDemo();
        obj.setAge(30);
        System.out.println(obj.getAge());  // 30

        try {
            obj.setAge(-5);
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());  // Caught: Invalid age
        }
    }
}
```

**Expected Output:**
```
30
Caught: Invalid age
```

**Step-by-Step Explanation:**

1. `setAge()` validates the input before assigning it to the field.
2. Invalid values throw an exception.

### Real-World Cases

- **Validation:** Setters validate email formats, age ranges, and required fields.
- **JavaBeans:** Setters are required for writable properties.
- **Builder Pattern:** Setters are replaced by builder methods.

---

## 9. Properties (JavaBeans)

### Definitions

**Core Definition:** A JavaBeans property is a private field exposed through public getter and setter methods following specific naming conventions.

**Technical Definition:** The JavaBeans specification defines a property as a private value accessed through getter and setter methods. For a property named `x`, the getter is `getX()` and the setter is `setX()`. For boolean properties, the getter may be `isX()`.

**Beginner-Friendly Explanation:** A JavaBean property is like a standardized way to expose data — the field is private, and the getter/setter methods follow a naming convention that tools can recognize.

### Purposes

- To provide a standardized way to expose object state
- To enable tools and frameworks to discover and manipulate properties
- To support data binding and UI components

### Syntax Rules and Structure

```java
public class Bean {
    private type propertyName;

    public type getPropertyName() {
        return propertyName;
    }

    public void setPropertyName(type value) {
        this.propertyName = value;
    }
}
```

**Syntax Rules:**

- Property names are derived from getter/setter names by removing the prefix and lowercasing the first letter.
- Boolean properties may use `is` instead of `get`.
- Both getter and setter must be `public`.

**Constraints and Limitations:**

- Properties are a convention, not a language feature.
- Read-only properties have a getter but no setter.
- Write-only properties have a setter but no getter.

### Annotated Code Example

```java
// File: JavaBeanDemo.java
public class JavaBeanDemo {
    private String name;
    private int age;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    public static void main(String[] args) {
        JavaBeanDemo bean = new JavaBeanDemo();
        bean.setName("Alice");
        bean.setAge(30);
        System.out.println(bean.getName() + " is " + bean.getAge());  // Alice is 30
    }
}
```

**Expected Output:**
```
Alice is 30
```

**Step-by-Step Explanation:**

1. `name` and `age` are private fields.
2. `getName()`/`setName()` and `getAge()`/`setAge()` follow the JavaBeans convention.
3. Tools like IDEs and frameworks can discover the `name` and `age` properties.

### Real-World Cases

- **UI Frameworks:** JavaFX and Swing use properties for data binding.
- **Spring Framework:** Spring beans expose properties for dependency injection.
- **Persistence:** JPA entities use JavaBeans properties for database mapping.

---

## 10. Immutability

### Definitions

**Core Definition:** An immutable object is an object whose state cannot be changed after it is created.

**Technical Definition:** Immutable classes are declared `final`, all fields are `private` and `final`, no mutator methods are provided, and mutable fields are defensively copied. Immutable objects are inherently thread-safe and can be shared freely.

**Beginner-Friendly Explanation:** An immutable object is like a printed book — once it's printed, you cannot change the words. If you want a different book, you create a new one.

### Purposes

- To ensure thread safety without synchronization
- To prevent unintended side effects
- To simplify reasoning about program state
- To enable safe use as keys in hash-based collections

### Syntax Rules and Structure

```java
public final class ImmutableClass {
    private final type field1;
    private final type field2;

    public ImmutableClass(type field1, type field2) {
        this.field1 = field1;
        this.field2 = field2;
    }

    public type getField1() { return field1; }
    public type getField2() { return field2; }
    // No setters!
}
```

**Syntax Rules:**

- Declare the class `final` to prevent subclassing.
- Declare all fields `private` and `final`.
- Do not provide mutator methods.
- For mutable fields, return defensive copies in getters.

**Constraints and Limitations:**

- Immutable objects can lead to increased memory usage.
- Creating truly immutable classes with mutable fields requires defensive copying.

### Annotated Code Example

```java
// File: ImmutableDemo.java
import java.util.Date;

public final class ImmutableDemo {
    private final String name;
    private final Date birthDate;

    public ImmutableDemo(String name, Date birthDate) {
        this.name = name;
        this.birthDate = new Date(birthDate.getTime());  // Defensive copy
    }

    public String getName() { return name; }

    public Date getBirthDate() {
        return new Date(birthDate.getTime());  // Defensive copy
    }

    public static void main(String[] args) {
        Date originalDate = new Date(90, 0, 1);  // Jan 1, 1990
        ImmutableDemo person = new ImmutableDemo("Alice", originalDate);

        originalDate.setYear(100);  // Does NOT affect person
        System.out.println(person.getBirthDate().getYear() + 1900);  // 1990

        person.getBirthDate().setYear(200);  // Does NOT affect person
        System.out.println(person.getBirthDate().getYear() + 1900);  // 1990
    }
}
```

**Expected Output:**
```
1990
1990
```

**Step-by-Step Explanation:**

1. The class is `final` with `private final` fields.
2. The constructor makes a defensive copy of the `Date`.
3. The getter returns a defensive copy.
4. External modifications to the original date or the returned copy do not affect the object.

### Real-World Cases

- **`String`:** Every "modification" creates a new `String`.
- **`Integer`, `Long`, `Double`:** Wrapper classes are immutable.
- **`LocalDate`, `LocalTime`:** The `java.time` API is immutable and thread-safe.
- **`BigDecimal`:** Used for precise monetary calculations.

---

## 11. Final Fields

### Definitions

**Core Definition:** A `final` field is a field whose value cannot be changed after initialization.

**Technical Definition:** A field declared `final` can be assigned exactly once, either at declaration or in a constructor. For reference types, `final` makes the reference immutable but does not make the referenced object immutable.

**Beginner-Friendly Explanation:** A `final` field is like a sealed envelope — once you put something in it, you cannot change it.

### Purposes

- To create constants
- To support immutability
- To enable safe publication of objects in concurrent programming

### Syntax Rules and Structure

```java
private final type fieldName = initialValue;

// Or initialized in constructor:
private final type fieldName;

public ClassName(type value) {
    this.fieldName = value;
}
```

**Syntax Rules:**

- `final` fields must be initialized exactly once.
- They can be initialized at declaration or in every constructor.
- `static final` fields are constants.

**Constraints and Limitations:**

- `final` on a reference does not make the referenced object immutable.
- `final` fields cannot be reassigned.

### Annotated Code Example

```java
// File: FinalFieldDemo.java
public class FinalFieldDemo {
    private final int constant = 100;
    private final String name;

    public FinalFieldDemo(String name) {
        this.name = name;  // Initialized in constructor
    }

    public static void main(String[] args) {
        FinalFieldDemo obj = new FinalFieldDemo("Alice");
        System.out.println(obj.constant);  // 100
        System.out.println(obj.name);       // Alice
    }
}
```

**Expected Output:**
```
100
Alice
```

**Step-by-Step Explanation:**

1. `constant` is initialized at declaration.
2. `name` is initialized in the constructor.
3. Both fields cannot be reassigned after initialization.

### Real-World Cases

- **Constants:** `Math.PI`, `Integer.MAX_VALUE`.
- **Immutable Classes:** All fields are `final`.
- **Singleton Pattern:** The instance field is `static final`.

---

## 12. Defensive Copying

### Definitions

**Core Definition:** Defensive copying is the practice of making copies of mutable objects to prevent external code from modifying internal state.

**Technical Definition:** Defensive copying involves copying mutable parameters before storing them and copying mutable fields before returning them. This prevents callers from modifying an object's internal state through references to its mutable components.

**Beginner-Friendly Explanation:** Defensive copying is like giving someone a photocopy of a document instead of the original — they can scribble on the copy, but your original remains safe.

### Purposes

- To protect internal state from external modification
- To prevent security vulnerabilities caused by mutable object exposure
- To maintain class invariants

### Syntax Rules and Structure

```java
// Defensive copy of constructor parameter
public ClassName(MutableType param) {
    this.field = new MutableType(param);  // Copy, don't store reference
}

// Defensive copy in getter
public MutableType getField() {
    return new MutableType(this.field);  // Return copy, not reference
}
```

**Syntax Rules:**

- Copy mutable parameters before storing them.
- Copy mutable fields before returning them.
- Use constructors or copy methods, not `clone()`, for defensive copying of untrusted inputs.

**Constraints and Limitations:**

- Defensive copying has a performance cost.
- Deep copying is required for objects with nested mutable fields.

### Annotated Code Example

```java
// File: DefensiveCopyDemo.java
import java.util.Date;

public class DefensiveCopyDemo {
    private final Date startDate;

    public DefensiveCopyDemo(Date startDate) {
        // Defensive copy: store a copy, not the original
        this.startDate = new Date(startDate.getTime());
    }

    public Date getStartDate() {
        // Defensive copy: return a copy, not the internal reference
        return new Date(startDate.getTime());
    }

    public static void main(String[] args) {
        Date original = new Date(1234567890000L);
        DefensiveCopyDemo obj = new DefensiveCopyDemo(original);

        // Modify the original — does NOT affect obj
        original.setTime(0);
        System.out.println("Original: " + original.getTime());  // 0
        System.out.println("Object:   " + obj.getStartDate().getTime());  // 1234567890000

        // Modify the returned copy — does NOT affect obj
        obj.getStartDate().setTime(0);
        System.out.println("Object after getter mutation: " + obj.getStartDate().getTime());  // 1234567890000
    }
}
```

**Expected Output:**
```
Original: 0
Object:   1234567890000
Object after getter mutation: 1234567890000
```

**Step-by-Step Explanation:**

1. The constructor makes a defensive copy of the `Date`.
2. The getter returns a defensive copy.
3. Modifying the original or the returned copy does not affect the object's internal state.

### Real-World Cases

- **`Period` Class:** Effective Java's `Period` class uses defensive copying for `Date` fields.
- **Collections:** Returning `Collections.unmodifiableList()` instead of the original list.
- **Security-Sensitive Code:** Defensive copying prevents malicious code from modifying internal state.

---

## 13. Encapsulation Best Practices

### Definitions

**Core Definition:** Encapsulation best practices are guidelines for designing classes that properly hide internal state while providing controlled access.

**Technical Definition:** Best practices derive from the SEI CERT Oracle Coding Standard and Effective Java. Key practices include declaring data members as private, providing wrapper methods, minimizing accessibility, using defensive copying, and favouring immutability.

**Beginner-Friendly Explanation:** Encapsulation best practices are like the rules for designing a secure building — keep private areas locked, provide controlled entry points, and don't give out keys to everyone.

### Purposes

- To ensure data integrity and security
- To maintain flexibility for future changes
- To create clean, maintainable APIs
- To prevent common encapsulation pitfalls

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Declare data members `private` | Prevents direct access and modification. |
| Use the most restrictive access | Start with `private` and only increase visibility when necessary. |
| Provide wrapper methods | Use getters/setters or domain methods to access private state. |
| Validate in setters | Check input before modifying state. |
| Defensive copying | Copy mutable parameters and return copies of mutable fields. |
| Prefer immutability | Immutable objects are thread-safe and simpler to reason about. |
| Avoid public fields | Except for constants (`public static final`). |
| Document the API | Use Javadoc to describe public methods and their contracts. |

**Syntax Rules:**

- Fields should be `private` unless there is a compelling reason otherwise.
- Getters and setters should only be provided when needed.
- Prefer domain methods (e.g., `deposit()`, `withdraw()`) over raw setters.

**Constraints and Limitations:**

- Over-encapsulation can lead to anemic domain models.
- Defensive copying has a performance cost.

### Annotated Code Example

```java
// File: BestPracticesEncapsulation.java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

public final class BestPracticesEncapsulation {
    private final String name;
    private final List<String> items;

    public BestPracticesEncapsulation(String name, List<String> items) {
        this.name = Objects.requireNonNull(name, "Name cannot be null");
        // Defensive copy of mutable list
        this.items = new ArrayList<>(items);
    }

    public String getName() {
        return name;
    }

    // Return an unmodifiable view instead of a copy
    public List<String> getItems() {
        return Collections.unmodifiableList(items);
    }

    // Domain method instead of raw setter
    public void addItem(String item) {
        if (item == null || item.isEmpty()) {
            throw new IllegalArgumentException("Item cannot be empty");
        }
        items.add(item);
    }

    public static void main(String[] args) {
        List<String> initialItems = new ArrayList<>();
        initialItems.add("Apple");
        BestPracticesEncapsulation obj = new BestPracticesEncapsulation("Cart", initialItems);

        initialItems.add("Banana");  // Does NOT affect obj
        System.out.println(obj.getItems());  // [Apple]

        obj.addItem("Cherry");
        System.out.println(obj.getItems());  // [Apple, Cherry]

        // obj.getItems().add("Date");  // UnsupportedOperationException
    }
}
```

**Expected Output:**
```
[Apple]
[Apple, Cherry]
```

**Step-by-Step Explanation:**

1. The class is `final`, fields are `private final`.
2. The constructor makes a defensive copy of the list.
3. `getItems()` returns an unmodifiable view.
4. `addItem()` is a domain method with validation, not a raw setter.

### Real-World Cases

- **`Collections.unmodifiableList`:** Returns a read-only view of a list.
- **`String` Class:** Entirely immutable; all "modifications" create new objects.
- **Domain-Driven Design:** Entities use domain methods instead of generic setters.

---

## References Links

- Controlling Access to Members of a Class — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html
- Java Language Specification, §6.6: Access Control - https://docs.oracle.com/javase/specs/jls/se21/html/jls-6.html#jls-6.6
- Properties — Oracle JavaBeans Tutorial - https://docs.oracle.com/javase/tutorial/javabeans/writing/properties.html
- Access Modifiers in Java — Baeldung - https://www.baeldung.com/java-access-modifiers
- SEI CERT Oracle Coding Standard — OBJ01-J: Declare data members as private and provide accessible wrapper methods - https://wiki.sei.cmu.edu/confluence/display/java/OBJ01-J.+Declare+data+members+as+private+and+provide+accessible+wrapper+methods
- SEI CERT Oracle Coding Standard — OBJ09-J: Defensively copy private mutable class members before returning their references - https://wiki.sei.cmu.edu/confluence/display/java/OBJ09-J.+Defensively+copy+private+mutable+class+members+before+returning+their+references
- SEI CERT Oracle Coding Standard — MET52-JG: Do not use the clone method to copy untrusted method parameters - https://wiki.sei.cmu.edu/confluence/display/java/MET52-JG.+Do+not+use+the+clone+method+to+copy+untrusted+method+parameters
- Effective Java (3rd Edition) by Joshua Bloch — Item 15: Minimize mutability
- Principles of Software Construction — Carnegie Mellon University - http://www.cs.cmu.edu/~charlie/courses/17-214/2019-fall/slides/20191001-invariants-immutabilility-and-testing.pdf
- Encapsulation — CodeSignal - https://codesignal.com/learn/courses/revisiting-oop-concepts/lessons/encapsulation
- Encapsulation in Java — Scaler Topics - https://www.scaler.com/topics/encapsulation-in-java/