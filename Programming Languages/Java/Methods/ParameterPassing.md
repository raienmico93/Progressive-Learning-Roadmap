# Java Parameter Passing: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Java parameter passing** is the mechanism by which arguments are transferred from a method caller to the method's parameters. Java **always** uses **pass-by-value**: the value of the argument is copied into the parameter variable. For primitives, the copy is the actual data value. For objects, the copy is the **reference** (the memory address) to the object—not the object itself.

### Technical Definition

When a method is invoked, the Java Virtual Machine creates a new activation frame containing freshly created parameter variables. The argument values from the call site are assigned to these parameter variables . For primitive types, the argument's value is copied directly. For reference types, the argument's value is a reference (a handle) to an object on the heap; this reference is copied, so both the caller's variable and the method's parameter point to the same object. However, reassigning the parameter to point to a different object does not affect the caller's variable . This is known as **pass-by-value with reference values**.

### Beginner-Friendly Explanation

Think of a reference as a piece of paper with a house's address written on it. When you pass an object to a method, you're handing over a **photocopy** of that paper. Both you and the method now have papers pointing to the same house. If the method paints the house (modifies the object), you'll see the change. But if the method tears up its paper and writes a new address on a fresh one (reassigns the parameter), your original paper is unaffected—you still have the old address .

### Key Characteristics

- **Always pass-by-value**: Java never passes references by reference; it passes reference values by value .
- **Primitives copy the value**: Changes to primitive parameters never affect the caller.
- **Objects copy the reference**: The method receives a copy of the reference, pointing to the same object.
- **Mutations escape**: Modifying the object's state (via setters, field access, or mutating methods) affects the caller's object .
- **Reassignment is local**: Pointing the parameter to a new object does not change the caller's reference .
- **Immutable objects**: For immutable types like `String`, `Integer`, and other wrapper classes, no mutation is possible, so only reassignment matters—and that never affects the caller .

### Prerequisites

- Basic Java syntax (variables, methods, classes).
- Understanding of primitive types vs. reference types.
- Familiarity with object references and the heap/stack model.

### Related Programming Areas

- **Object-Oriented Programming**: Encapsulation and method contracts.
- **Immutability**: Designing classes whose state cannot change after construction.
- **Concurrency**: Shared mutable state across threads.
- **API Design**: Defensive copying and parameter validation.

### Core Concepts / Features

1. Java Pass-by-Value Semantics
2. Primitive Arguments vs. Object Reference Arguments
3. Object References as Values (Copy of the Reference)
4. Effect of Mutations (State vs. Reference Pointer)
5. Immutable Objects (like `String`) and Method Calls


## Core Concept 1: Java Pass-by-Value Semantics

### Definitions

**Core Definition**: Pass-by-value means that when a method is called, the value of each argument is copied into the corresponding parameter variable. The method operates on these copies, not on the caller's original variables.

**Technical Definition**: The Java programming language always uses call by value . When a method is invoked, a new activation frame is created, and the argument values are assigned to freshly created parameter variables . For primitives, the value itself is copied. For objects, the value is a reference (handle) to the object; the reference is copied, but the object is shared. Because the parameter is a copy, reassigning it (e.g., `param = new Object()`) has no effect on the caller's variable .

**Beginner-Friendly Explanation**: Imagine you have a piece of paper with the number 5 written on it. You photocopy it and give the copy to a friend. If your friend erases the 5 and writes 10, your original paper still says 5. That's pass-by-value for primitives. For objects, the paper has an address on it. Your friend can go to that address and rearrange the furniture (modify the object), but if they write a different address on their copy, your paper still has the original address.

### Purposes

- To ensure that method calls cannot unexpectedly change the caller's primitive variables.
- To provide a clear, predictable model for how data flows into methods.
- To explain why object mutations are visible to the caller while reference reassignment is not.
- To guide the design of methods that either mutate or preserve their inputs.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Method declaration
ReturnType methodName(ParameterType parameterName) {
    // parameterName is a local copy of the argument's value
}

// Method invocation
methodName(argumentValue);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ParameterType` | The declared type of the parameter (primitive or reference). |
| `parameterName` | The local variable inside the method holding the copied value. |
| `argumentValue` | The value passed at the call site; its value is copied. |

**Syntax Rules:**

- The number, type, and order of arguments must match the parameters.
- The parameter variable is local to the method and is destroyed when the method returns.
- Assignments to the parameter variable never affect the caller's argument.
- For reference types, the parameter holds a copy of the reference, so both point to the same object.

**Constraints and Limitations:**

- Java does **not** support pass-by-reference; there is no way to make a method change which object a caller's variable points to .
- A `swap` method for objects cannot work in Java without using a holder object (e.g., an array or wrapper class) .
- Primitive values are copied directly; object references are copied, but the objects themselves are not copied.

### Annotated Complete Code Examples

**Example 1: Pass-by-Value with Primitives**

```java
/**
 * Demonstrates that primitive arguments are passed by value.
 */
public class PrimitivePassByValue {

    // The parameter 'x' is a copy of the argument
    public static void modify(int x) {
        x = x + 10; // Changes the local copy only
        System.out.println("Inside method: x = " + x);
    }

    public static void main(String[] args) {
        int original = 5;
        System.out.println("Before call: original = " + original);

        modify(original); // The value 5 is copied into x

        System.out.println("After call: original = " + original); // Still 5
    }
}
```

**Expected Output:**

```
Before call: original = 5
Inside method: x = 15
After call: original = 5
```

**Why This Output Occurs:**
- `modify(original)` passes the value `5` by copying it into the parameter `x`.
- Inside the method, `x = x + 10` changes the local copy to `15`.
- The caller's variable `original` remains `5` because it was never modified—only its value was copied.

**Step-by-Step Setup Guide:**
1. Create `PrimitivePassByValue.java`.
2. Compile with `javac PrimitivePassByValue.java`.
3. Run with `java PrimitivePassByValue`.
4. Observe the output.

### Real-World Cases

- **Calculations**: Utility methods like `Math.max(a, b)` do not modify their arguments.
- **Validation**: Methods that check a value without changing it.
- **Logging**: Methods that print a value without affecting the caller.

### References

- Oracle Java Tutorials – Passing Information to a Method or a Constructor - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Java Language Specification – Method Invocation (Create Frame) - https://docs.oracle.com/javase/specs/jls/se16/html/jls-15.html
- GitHub – Pass by Value vs Reference - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/12-pass-by-value-vs-reference.md


## Core Concept 2: Primitive Arguments vs. Object Reference Arguments

### Definitions

**Core Definition**: Primitive arguments pass a copy of the actual data value; object reference arguments pass a copy of the reference (memory address) to the object.

**Technical Definition**: For primitive types (`int`, `double`, `boolean`, `char`, etc.), the argument's value is copied directly into the parameter. Changes to the parameter never affect the caller. For reference types (classes, interfaces, arrays), the argument's value is a reference to an object on the heap. This reference is copied into the parameter, so both the caller's variable and the method's parameter point to the same object. The object itself is not copied .

**Beginner-Friendly Explanation**: Passing a primitive is like giving someone a photocopy of a photo—they can scribble on their copy, but your original is safe. Passing an object reference is like giving someone a photocopy of a map to your house—they can go to your house and rearrange the furniture, but if they write a different address on their map, your map still points to your house.

### Purposes

- To understand the different behaviors of primitive and reference parameters.
- To explain why primitive changes are local but object mutations are visible.
- To guide the choice between primitive and object parameter types.
- To avoid the common misconception that Java passes objects by reference.

### Syntax Rules and Structure

**Comparison Table:**

| Aspect | Primitive Argument | Object Reference Argument |
|--------|-------------------|--------------------------|
| Value passed | The data value itself | A copy of the reference (address) |
| Parameter holds | Independent copy of the value | Copy of the reference to the same object |
| Modifying parameter | Affects only the local copy | Affects the local copy of the reference (reassignment) |
| Modifying object state | N/A (no object) | Visible to the caller (same object) |
| Example types | `int`, `double`, `boolean` | `String`, `ArrayList`, custom classes |

**Syntax Rules:**

- Primitive parameters are declared with primitive types (e.g., `int x`).
- Reference parameters are declared with class/interface types (e.g., `StringBuilder sb`).
- Both are passed by value; the difference is what the value represents.
- For reference types, the parameter is a copy of the reference, not a copy of the object.

**Constraints and Limitations:**

- You cannot pass a primitive where a reference is expected without autoboxing.
- You cannot change the caller's reference by reassigning the parameter.
- Object mutations made through the parameter affect the caller's object.

### Annotated Complete Code Examples

**Example 1: Primitive vs. Object Reference Arguments**

```java
/**
 * Demonstrates the difference between primitive and reference arguments.
 */
public class PrimitiveVsReference {

    // Primitive parameter: changes are local
    public static void changePrimitive(int value) {
        value = 100;
        System.out.println("Inside changePrimitive: " + value);
    }

    // Reference parameter: can mutate the object
    public static void changeObjectState(StringBuilder sb) {
        sb.append(" World"); // Mutates the shared object
        System.out.println("Inside changeObjectState: " + sb);
    }

    // Reference parameter: reassignment is local
    public static void reassignReference(StringBuilder sb) {
        sb = new StringBuilder("New object"); // Local reassignment only
        System.out.println("Inside reassignReference: " + sb);
    }

    public static void main(String[] args) {
        int num = 5;
        System.out.println("Before changePrimitive: " + num);
        changePrimitive(num);
        System.out.println("After changePrimitive: " + num); // Still 5

        StringBuilder builder = new StringBuilder("Hello");
        System.out.println("\nBefore changeObjectState: " + builder);
        changeObjectState(builder);
        System.out.println("After changeObjectState: " + builder); // "Hello World"

        System.out.println("\nBefore reassignReference: " + builder);
        reassignReference(builder);
        System.out.println("After reassignReference: " + builder); // Still "Hello World"
    }
}
```

**Expected Output:**

```
Before changePrimitive: 5
Inside changePrimitive: 100
After changePrimitive: 5

Before changeObjectState: Hello
Inside changeObjectState: Hello World
After changeObjectState: Hello World

Before reassignReference: Hello World
Inside reassignReference: New object
After reassignReference: Hello World
```

**Why This Output Occurs:**
- `changePrimitive(num)` copies the value `5`; the local change to `100` does not affect `num`.
- `changeObjectState(builder)` copies the reference; both `builder` and `sb` point to the same `StringBuilder`. Appending mutates the shared object, so the change is visible.
- `reassignReference(builder)` copies the reference; reassigning `sb` to a new object changes only the local copy. The caller's `builder` still points to the original object.

### Real-World Cases

- **Mutating collections**: Passing a `List` to a method that adds elements affects the caller's list.
- **String manipulation**: Passing a `String` to a method cannot change the caller's string because `String` is immutable.
- **Builder patterns**: Passing a `StringBuilder` allows methods to append to the same builder.
- **Swap methods**: Cannot work in Java with simple reference parameters; require holder objects.

### References

- Oracle Java Tutorials – Passing Information to a Method or a Constructor - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- GitHub – Pass by Value vs Reference - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/12-pass-by-value-vs-reference.md
- Stack Overflow – Pass-by-value explanation - https://stackoverflow.com/revisions/5fa950fb-011a-493e-bbb8-4ce11f574422/view-source


## Core Concept 3: Object References as Values (Copy of the Reference)

### Definitions

**Core Definition**: When an object is passed to a method, the **reference** to that object is copied, not the object itself. The method receives its own copy of the reference, which points to the same object on the heap.

**Technical Definition**: Java passes objects as references passed by value . The parameter variable inside the method is a distinct variable that holds a copy of the reference value from the call site. Both the caller's variable and the method's parameter refer to the same object in memory. However, because the reference is copied, reassigning the parameter to a different object does not change the caller's variable .

**Beginner-Friendly Explanation**: Think of a reference as a remote control for a TV. When you pass an object to a method, you give the method a **copy** of the remote. Both remotes control the same TV. The method can change the channel (mutate the object), but if the method throws away its remote and picks up a different one (reassigns the parameter), your remote still controls the original TV.

### Purposes

- To explain why object mutations are visible to the caller.
- To explain why reference reassignment is not visible to the caller.
- To provide a mental model for understanding Java's parameter passing.
- To guide the design of methods that mutate or reassign object references.

### Syntax Rules and Structure

**Mental Model:**

```java
// Caller's variable
Object ref = new Object();

// Method parameter
void method(Object param) {
    // param is a COPY of ref
    // Both point to the same object
}
```

**Diagram:**

```
Caller's variable: ref ---> [Object A]
Method's parameter: param ---> [Object A]  (same object)

After reassignment inside method:
Caller's variable: ref ---> [Object A]
Method's parameter: param ---> [Object B]  (different object)
```

**Syntax Rules:**

- The parameter variable is a distinct local variable.
- The reference value is copied at the call site.
- Mutating the object through the parameter affects the caller's object.
- Reassigning the parameter does not affect the caller's variable.

**Constraints and Limitations:**

- There is no way to make a method change which object a caller's variable points to.
- A swap method for object references cannot work without a holder object .
- The copied reference is a shallow copy; the object's fields are not copied.

### Annotated Complete Code Examples

**Example 1: Copy of the Reference**

```java
/**
 * Demonstrates that the reference is copied, not the object.
 */
public class ReferenceCopyDemo {

    static class Person {
        String name;
        Person(String name) { this.name = name; }
        @Override
        public String toString() { return "Person[" + name + "]"; }
    }

    // Mutates the object through the copied reference
    public static void rename(Person p) {
        p.name = "Alice"; // Mutates the shared object
    }

    // Reassigns the local copy of the reference
    public static void replace(Person p) {
        p = new Person("Bob"); // Changes the local copy only
    }

    public static void main(String[] args) {
        Person person = new Person("Charlie");

        System.out.println("Before rename: " + person);
        rename(person);
        System.out.println("After rename: " + person); // Alice

        System.out.println("\nBefore replace: " + person);
        replace(person);
        System.out.println("After replace: " + person); // Still Alice
    }
}
```

**Expected Output:**

```
Before rename: Person[Charlie]
After rename: Person[Alice]

Before replace: Person[Alice]
After replace: Person[Alice]
```

**Why This Output Occurs:**
- `rename(person)` passes a copy of the reference. Both `person` and `p` point to the same `Person` object. Changing `p.name` mutates the shared object, so the caller sees "Alice".
- `replace(person)` passes a copy of the reference. Reassigning `p = new Person("Bob")` changes only the local copy. The caller's `person` still points to the original object ("Alice").

### Real-World Cases

- **Mutating collections**: A method that adds items to a `List` parameter affects the caller's list.
- **Reassigning parameters**: A method that creates a new list and reassigns the parameter does not affect the caller.
- **Holder objects**: To return multiple values, pass an array or use a wrapper class.

### References

- Stack Overflow – Java passes objects as references passed by value - https://stackoverflow.com/revisions/93b32669-c4a8-477b-b8eb-74bcfa479435/view-source
- GitHub – Pass by Value vs Reference - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/12-pass-by-value-vs-reference.md
- Core Java Volume I – ParamTest (Listing 4.4) - https://archive.org/stream/core-java-volume-i-fundamentals-11th-edition/core-java-volume-i-fundamentals-11th-edition_djvu.txt


## Core Concept 4: Effect of Mutations (Modifying Object State vs. Changing the Reference Pointer)

### Definitions

**Core Definition**: Modifying an object's state through a parameter changes the shared object (visible to the caller). Changing the parameter's reference pointer to a different object does **not** affect the caller's variable.

**Technical Definition**: Because the parameter holds a copy of the reference, both the caller and the method can access and mutate the same object. Mutations to the object's fields (via setters, direct field access, or mutating methods) are visible to the caller. However, reassigning the parameter variable (e.g., `param = new Object()`) changes only the local copy of the reference; the caller's variable continues to point to the original object .

**Beginner-Friendly Explanation**: If you and a friend both have keys to the same house, your friend can repaint the walls (mutate the object), and you'll see the change. But if your friend throws away their key and gets a key to a different house (reassigns the reference), your key still opens the original house.

### Purposes

- To understand when changes made inside a method are visible to the caller.
- To avoid unintended side effects from object mutations.
- To design methods that either mutate (and document it) or preserve their inputs.
- To explain why swap methods fail in Java.

### Syntax Rules and Structure

**Mutation (Visible to Caller):**

```java
void mutate(List<String> list) {
    list.add("new item"); // Visible to caller
}
```

**Reassignment (NOT Visible to Caller):**

```java
void reassign(List<String> list) {
    list = new ArrayList<>(); // Local only
    list.add("new item"); // Not visible to caller
}
```

**Comparison:**

| Action | Effect on Caller's Object |
|--------|--------------------------|
| `list.add(item)` | Visible (mutates shared object) |
| `list.set(index, item)` | Visible (mutates shared object) |
| `list = new ArrayList<>()` | Not visible (reassigns local copy) |
| `list.clear()` | Visible (mutates shared object) |

**Syntax Rules:**

- Mutating methods (e.g., `add`, `set`, `clear`) affect the shared object.
- Reassigning the parameter (e.g., `param = new ...`) affects only the local copy.
- To return a modified object, return it from the method: `list = modify(list)`.
- To avoid mutation, copy the object inside the method before modifying.

**Constraints and Limitations:**

- Unintended mutation of parameters is a common source of bugs.
- Immutable objects sidestep the mutation issue entirely.
- Defensive copying prevents callers from modifying internal collections.

### Annotated Complete Code Examples

**Example 1: Mutation vs. Reassignment**

```java
import java.util.*;

/**
 * Demonstrates the difference between mutating an object and reassigning a reference.
 */
public class MutationVsReassignment {

    // Mutates the list (visible to caller)
    public static void addToList(List<String> list) {
        list.add("added"); // Mutates shared object
    }

    // Reassigns the local reference (NOT visible to caller)
    public static void replaceList(List<String> list) {
        list = new ArrayList<>(); // Local reassignment
        list.add("replaced");
    }

    public static void main(String[] args) {
        List<String> original = new ArrayList<>();
        original.add("initial");

        System.out.println("Before addToList: " + original);
        addToList(original);
        System.out.println("After addToList: " + original); // [initial, added]

        System.out.println("\nBefore replaceList: " + original);
        replaceList(original);
        System.out.println("After replaceList: " + original); // Still [initial, added]
    }
}
```

**Expected Output:**

```
Before addToList: [initial]
After addToList: [initial, added]

Before replaceList: [initial, added]
After replaceList: [initial, added]
```

**Why This Output Occurs:**
- `addToList(original)` passes a copy of the reference. Both `original` and `list` point to the same `ArrayList`. `list.add("added")` mutates the shared object, so the caller sees the new item.
- `replaceList(original)` passes a copy of the reference. `list = new ArrayList<>()` reassigns the local copy to a new object. The caller's `original` still points to the first list, so the "replaced" item is not visible.

### Real-World Cases

- **Defensive copying**: Constructors copy mutable parameters to prevent external modification.
- **Builder methods**: Mutate the builder and return `this` for chaining.
- **Swap functions**: Cannot work in Java; use holder objects or return values.
- **Collection operations**: Methods like `Collections.sort(list)` mutate the list in place.

### References

- Stack Overflow – Java passes objects as references passed by value - https://stackoverflow.com/revisions/93b32669-c4a8-477b-b8eb-74bcfa479435/view-source
- GitHub – Pass by Value vs Reference - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/12-pass-by-value-vs-reference.md
- Core Java Volume I – ParamTest - https://archive.org/stream/core-java-volume-i-fundamentals-11th-edition/core-java-volume-i-fundamentals-11th-edition_djvu.txt


## Core Concept 5: Immutable Objects (like `String`) and Method Calls

### Definitions

**Core Definition**: An immutable object is an object whose state cannot be changed after construction. When passed to a method, an immutable object's contents cannot be modified; only the reference can be reassigned (which is local to the method).

**Technical Definition**: The `String` class is immutable, meaning that once a `String` object is created, its value cannot be changed . Methods that appear to modify a `String` (e.g., `toUpperCase()`, `substring()`) actually create and return a new `String` object. When an immutable object is passed to a method, the method cannot mutate its contents. Any reassignment of the parameter is local and does not affect the caller .

**Beginner-Friendly Explanation**: An immutable object is like a printed book. You can't change the words on the page. If you want a different book, you have to print a new one. When you pass a `String` to a method, the method can read it, but it can't change it—it can only get a new string back.

### Purposes

- To understand why `String` and wrapper types behave differently in method calls.
- To explain why mutations never escape for immutable types.
- To guide the design of immutable classes for thread safety and predictability.
- To avoid confusion when passing immutable objects to methods.

### Syntax Rules and Structure

**Immutable Types in Java:**

| Type | Description |
|------|-------------|
| `String` | Immutable character sequence |
| `Integer`, `Long`, `Double`, etc. | Immutable wrapper classes |
| `BigInteger`, `BigDecimal` | Immutable numeric types |
| `LocalDate`, `LocalTime`, `Instant` | Immutable date-time types |

**Method Call Behavior:**

```java
String s = "hello";
void modify(String str) {
    str = str.toUpperCase(); // Reassigns local copy; creates new String
}
// s remains "hello"
```

**Syntax Rules:**

- Immutable objects cannot be mutated through a parameter.
- Methods that "modify" immutable objects return new objects.
- Reassigning an immutable parameter is local and does not affect the caller.
- To return a modified value, return the new object: `s = s.toUpperCase()`.

**Constraints and Limitations:**

- You cannot modify a `String` in place; you must create a new one.
- Passing an immutable object to a method that expects to modify it will not work.
- Immutable objects are inherently thread-safe, which is a major advantage.

### Annotated Complete Code Examples

**Example 1: Immutable String in Method Calls**

```java
/**
 * Demonstrates that immutable String objects cannot be mutated through methods.
 */
public class ImmutableStringDemo {

    // Attempts to modify the String parameter
    public static void modifyString(String str) {
        str = str.toUpperCase(); // Creates a new String, reassigns local copy
        System.out.println("Inside modifyString: " + str);
    }

    // Returns a new modified String
    public static String returnModifiedString(String str) {
        return str.toUpperCase(); // Returns new String
    }

    public static void main(String[] args) {
        String original = "hello";

        System.out.println("Before modifyString: " + original);
        modifyString(original);
        System.out.println("After modifyString: " + original); // Still "hello"

        System.out.println("\nBefore returnModifiedString: " + original);
        String modified = returnModifiedString(original);
        System.out.println("After returnModifiedString: " + original); // Still "hello"
        System.out.println("Returned value: " + modified); // "HELLO"
    }
}
```

**Expected Output:**

```
Before modifyString: hello
Inside modifyString: HELLO
After modifyString: hello

Before returnModifiedString: hello
After returnModifiedString: hello
Returned value: HELLO
```

**Why This Output Occurs:**
- `modifyString(original)` passes a copy of the reference. `str = str.toUpperCase()` creates a new `String` and reassigns the local copy. The caller's `original` still points to `"hello"`.
- `returnModifiedString(original)` also creates a new `String`, but returns it. The caller assigns the returned value to `modified`, so `modified` is `"HELLO"` while `original` remains `"hello"`.

### Real-World Cases

- **String manipulation**: Methods like `trim()`, `replace()`, `substring()` return new strings.
- **Wrapper classes**: `Integer.valueOf()`, `Double.parseDouble()` return new instances.
- **Date-time API**: `LocalDate.plusDays()` returns a new `LocalDate`.
- **Thread safety**: Immutable objects can be shared freely across threads.

### References

- Oracle Java Tutorials – Strings - https://docs.oracle.com/javase/tutorial/java/data/strings.html
- GitHub – Immutable Classes - https://github.com/learn-co-curriculum/java-mod-1-immutable-classes
- Oracle Java API – String (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html


## References

- Oracle Java Tutorials – Passing Information to a Method or a Constructor - https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html
- Java Language Specification – Method Invocation (Create Frame) - https://docs.oracle.com/javase/specs/jls/se16/html/jls-15.html
- Oracle Java Tutorials – Strings - https://docs.oracle.com/javase/tutorial/java/data/strings.html
- Oracle Java API – String (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html
- GitHub – Pass by Value vs Reference - https://raw.githubusercontent.com/CamlineKe/software-engineering-notes/refs/heads/main/java/notes/03-oop-depth/12-pass-by-value-vs-reference.md
- GitHub – Immutable Classes - https://github.com/learn-co-curriculum/java-mod-1-immutable-classes
- Stack Overflow – Java passes objects as references passed by value - https://stackoverflow.com/revisions/93b32669-c4a8-477b-b8eb-74bcfa479435/view-source
- Stack Overflow – Pass-by-value explanation - https://stackoverflow.com/revisions/5fa950fb-011a-493e-bbb8-4ce11f574422/view-source
- Core Java Volume I – ParamTest (Listing 4.4) - https://archive.org/stream/core-java-volume-i-fundamentals-11th-edition/core-java-volume-i-fundamentals-11th-edition_djvu.txt
- Oracle Help Center – Handling Parameters - https://docs.oracle.com/cd/B31017%5F01/web.1013/b28221/usclient012.htm
- Oracle Help Center – Enterprise JavaBeans Parameter Passing - https://docs.oracle.com/cd/A69464_01/nt_815/java.815/a64683/ejb5.htm