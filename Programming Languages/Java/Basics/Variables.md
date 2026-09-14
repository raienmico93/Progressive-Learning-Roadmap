# Java Variables and Constants: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Java variables are named storage locations in memory that hold values of a specified data type. Constants are variables whose values cannot be changed after initialization. Together, they form the fundamental mechanism by which Java programs store, manipulate, and share state.

**Technical Definition:** According to the Java Language Specification, a *variable* is a storage location and has an associated *type*, sometimes called its *compile-time type*. Every variable must be declared before use. A *final variable* may only be assigned once; if it is also of primitive type or type `String` and initialized with a constant expression, it is termed a *constant variable*.

**Beginner-Friendly Explanation:** Think of a variable as a labeled box where your program keeps information. The label is the variable name, the size of the box is determined by the data type, and the contents are the value. A constant is a box that is sealed shut after you put something in it—you can look inside, but you can never swap the contents.

### Key Characteristics

- **Statically Typed:** Every variable’s type is known at compile time; type mismatches are compile-time errors.
- **Explicit Declaration Required:** Variables must be declared before they can be used.
- **Scope Determines Visibility:** Where a variable is declared determines which parts of the program can access it.
- **Default Values for Fields:** Instance and static variables receive default values; local variables do not.
- **Finality for Immutability:** The `final` keyword prevents reassignment, though the referenced object (if any) may remain mutable.

### Prerequisites

Before mastering variables and constants, you should understand:
- Basic Java program structure (classes, methods, `main`)
- Primitive data types and reference types
- Compilation and execution workflow of Java programs
- Basic object-oriented concepts (classes, instances, fields)

### Related Programming Areas

| Area | Explanation |
|------|-------------|
| **Memory Management** | Variables occupy stack or heap memory; understanding this clarifies lifetime and garbage collection behavior. |
| **Object-Oriented Design** | Instance variables define object state; static variables define class-level state. |
| **Concurrency** | Static and instance fields shared across threads require synchronization; final fields have safe publication guarantees. |
| **API Design** | Public constants communicate intent and prevent misuse; `public static final` fields are standard API practice. |
| **JVM Internals** | Constant variables are inlined at compile time, affecting binary compatibility. |

---

## Core Concepts / Key Features

### 1. Variable Lifecycle

#### Definitions

**Core Definition:** The variable lifecycle encompasses the three sequential phases a variable undergoes: declaration, initialization, and assignment/reassignment.

**Technical Definition:** Declaration introduces the variable name and type. Initialization is the first assignment that gives the variable a value. Reassignment updates an already-initialized variable (except `final` variables, which cannot be reassigned).

**Beginner-Friendly Explanation:** First you announce that a box exists (declaration), then you put something in it for the first time (initialization), then you might replace what’s inside (reassignment).

#### Sub-features

##### Variable Declaration (Syntax and Data Types)

**Definitions:**
- **Core Definition:** Declaration creates a named variable of a specified type in the current scope.
- **Technical Definition:** A local variable declaration statement consists of a type followed by a declarator list; each declarator may include an initializer.
- **Beginner Explanation:** You tell Java the name of your box and what kind of things it can hold.

**Purposes:**
- **To** reserve memory space for storing a value.
- **To** associate a human-readable name with a storage location.
- **To** enforce type safety by specifying what values are permitted.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
type variableName;
type variableName = initialValue;
type var1, var2, var3;
type var1 = init1, var2 = init2;
```

**Component Breakdown:**
- `type`: A primitive type (`int`, `double`, `boolean`, etc.) or a reference type (`String`, `ArrayList`, or any class/interface).
- `variableName`: A valid Java identifier following naming conventions.
- `initialValue`: An expression assignable to the declared type.
- `;`: Statement terminator.

**Syntax Rules:**
- Variable names are case-sensitive.
- Names must begin with a letter, `$`, or `_`; subsequent characters may include digits.
- Reserved keywords cannot be used as variable names.
- Convention: begin with a lowercase letter; use camelCase for multi-word names.

**Constraints and Limitations:**
- Local variables have no default value and must be definitely assigned before use.
- Instance and static variables receive default values if not explicitly initialized.
- The type cannot be omitted (except with `var`, discussed later).

**Annotated Code Examples:**

```java
// Example 1: Declaring primitive and reference types
public class DeclarationDemo {
    public static void main(String[] args) {
        // Declaration without initialization (local variable)
        int counter;              // Declares an int named counter
        String message;           // Declares a String reference named message
        
        // Declaration with initialization
        int maxSize = 100;        // Declares and initializes an int
        double ratio = 3.14159;   // double literal
        boolean isActive = true;  // boolean literal
        char grade = 'A';         // char literal
        
        // Multiple declarations on one line
        int x = 1, y = 2, z = 3;  // All initialized
        
        System.out.println("maxSize: " + maxSize);
        System.out.println("ratio: " + ratio);
        System.out.println("isActive: " + isActive);
        System.out.println("grade: " + grade);
        System.out.println("x+y+z: " + (x + y + z));
    }
}
```

**Expected Output:**
```
maxSize: 100
ratio: 3.14159
isActive: true
grade: A
x+y+z: 6
```

**Why This Output:** Each variable holds the literal value assigned to it. The `+` operator concatenates strings and numeric values; arithmetic is performed before concatenation due to parentheses.

**Execution Flow:** The JVM allocates stack space for local variables, assigns the literal values, then executes `System.out.println` calls sequentially.

**Real-World Case:** A banking application declares `double accountBalance;` and `String accountHolder;` to represent the essential state of a bank account object.

---

##### Variable Initialization (First-Time Assignment)

**Definitions:**
- **Core Definition:** Initialization is the assignment of a value to a variable for the first time after declaration.
- **Technical Definition:** A variable is initialized when an assignment expression targets it while it is definitely unassigned.
- **Beginner Explanation:** Putting something in the box for the first time.

**Purposes:**
- **To** provide a meaningful starting value for computation.
- **To** satisfy Java’s definite assignment rules for local variables.
- **To** establish object state in constructors.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
type variableName = expression;
variableName = expression;   // standalone initialization
```

**Syntax Rules:**
- The initializer expression must be assignable to the declared type (assignment compatibility).
- For local variables, the initializer must appear before any use of the variable.
- Instance and static variables are initialized to default values automatically.

**Constraints and Limitations:**
- Local variables are *not* automatically initialized to default values.
- The compiler performs *definite assignment analysis* to ensure a variable cannot be read before initialization.

**Annotated Code Examples:**

```java
// Example 2: Initialization requirements for local variables
public class InitializationDemo {
    public static void main(String[] args) {
        // Correct: initialized at declaration
        int initializedAtDeclaration = 42;
        System.out.println("Value: " + initializedAtDeclaration);
        
        // Correct: declared then initialized before use
        int declaredThenInitialized;
        declaredThenInitialized = 100;  // First assignment
        System.out.println("Value: " + declaredThenInitialized);
        
        // INCORRECT: This would cause a compile-time error
        // int neverInitialized;
        // System.out.println(neverInitialized);  // error: variable might not have been initialized
    }
}
```

**Expected Output:**
```
Value: 42
Value: 100
```

**Why This Output:** Both variables are assigned before being read. The commented code would fail to compile because Java’s definite assignment rules forbid reading a local variable before initialization.

**Real-World Case:** In a method that calculates shipping cost, a local `double totalCost;` must be initialized (e.g., `totalCost = basePrice + tax;`) before being returned.

---

##### Variable Assignment and Reassignment

**Definitions:**
- **Core Definition:** Assignment updates the value stored in a variable after it has already been initialized.
- **Technical Definition:** An assignment expression stores the value of the right-hand operand into the variable denoted by the left-hand operand, provided the variable is not `final`.
- **Beginner Explanation:** Replacing the contents of the box with something else.

**Purposes:**
- **To** update program state as computation progresses.
- **To** accumulate results (e.g., `sum += value`).
- **To** implement counters and loops.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
variableName = expression;
variableName op= expression;   // compound assignment
```

**Syntax Rules:**
- The variable must already be declared and initialized (for local variables).
- The expression must be assignment-compatible with the variable’s type.
- Compound operators (`+=`, `-=`, `*=`, `/=`, `%=`, etc.) perform the operation and assign the result.

**Constraints and Limitations:**
- `final` variables cannot be reassigned.
- Assignment expressions themselves have a value (the assigned value), enabling chaining like `a = b = c = 0;`.

**Annotated Code Examples:**

```java
// Example 3: Reassignment and compound assignment
public class ReassignmentDemo {
    public static void main(String[] args) {
        int score = 10;
        System.out.println("Initial score: " + score);
        
        score = 25;                    // Simple reassignment
        System.out.println("After reassignment: " + score);
        
        score += 5;                    // Compound assignment: score = score + 5
        System.out.println("After += 5: " + score);
        
        score *= 2;                    // score = score * 2
        System.out.println("After *= 2: " + score);
        
        // Chained assignment
        int a, b, c;
        a = b = c = 99;                // Assigns 99 to c, then b, then a
        System.out.println("a=" + a + " b=" + b + " c=" + c);
    }
}
```

**Expected Output:**
```
Initial score: 10
After reassignment: 25
After += 5: 30
After *= 2: 60
a=99 b=99 c=99
```

**Why This Output:** Each assignment replaces the prior value. Compound operators combine arithmetic with assignment. Chained assignment works because the assignment expression evaluates to the assigned value.

**Real-World Case:** A game loop tracks player score: `playerScore += pointsEarned;` after each enemy defeat.

---

### 2. Types of Variables (By Scope and Memory)

#### Definitions

**Core Definition:** Java classifies variables by where they are declared and how they are stored in memory, which determines their scope, lifetime, and default values.

**Technical Definition:** The JLS recognizes four kinds of variables: local variables, instance variables (non-static fields), static variables (class variables), and parameters.

**Beginner-Friendly Explanation:** Variables can live in different “neighborhoods” of your program—some belong to the whole class, some to individual objects, some just to a single method, and some are passed in as information.

#### Sub-features

##### Local Variables

**Definitions:**
- **Core Definition:** Local variables are variables declared inside a method, constructor, or block.
- **Technical Definition:** A local variable is a variable declared by a local variable declaration statement within a block (§14.4).
- **Beginner Explanation:** Temporary storage that exists only while a method or block is running.

**Purposes:**
- **To** hold temporary computation results within a method.
- **To** avoid polluting class-level state with method-specific data.
- **To** implement loop counters and intermediate values.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
type variableName = initializer;
```
(Declared inside a method, constructor, or block.)

**Syntax Rules:**
- Must be definitely assigned before use.
- Cannot use access modifiers (`public`, `private`, etc.).
- Scope extends from declaration to the end of the enclosing block.

**Constraints and Limitations:**
- No default values; must be explicitly initialized.
- Not accessible outside the declaring block.
- Stored on the stack (for primitives) or stack reference to heap (for objects).

**Annotated Code Examples:**

```java
// Example 4: Local variable scope
public class LocalVariableDemo {
    public static void main(String[] args) {
        int outer = 10;  // Local to main
        
        if (outer > 5) {
            int inner = 20;  // Local to the if-block
            System.out.println("Inner: " + inner);
            System.out.println("Outer accessible: " + outer);
        }
        
        // System.out.println(inner);  // ERROR: inner not in scope
        
        for (int i = 0; i < 3; i++) {  // i is local to the for loop
            System.out.println("i = " + i);
        }
        // System.out.println(i);  // ERROR: i not in scope
    }
}
```

**Expected Output:**
```
Inner: 20
Outer accessible: 10
i = 0
i = 1
i = 2
```

**Why This Output:** `outer` is visible throughout `main`. `inner` is visible only inside the `if` block. `i` is visible only inside the `for` loop. Attempts to access them outside their scope would cause compile errors.

**Real-World Case:** A method that parses a date string uses several local `int` variables (`year`, `month`, `day`) that are discarded once the method returns.

---

##### Instance Variables (Non-Static Fields)

**Definitions:**
- **Core Definition:** Instance variables are fields declared without the `static` keyword; each object has its own copy.
- **Technical Definition:** Non-static fields are members of a class that store object-specific state.
- **Beginner Explanation:** Information that each individual object remembers about itself.

**Purposes:**
- **To** represent the unique state of each object.
- **To** persist data across method calls on the same object.
- **To** enable object-oriented encapsulation of state.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
accessModifier type variableName = initializer;
```
(Declared inside a class but outside any method.)

**Syntax Rules:**
- May have access modifiers (`private`, `public`, `protected`, or package-private).
- Automatically initialized to default values if no initializer.
- Accessed via `objectName.fieldName` or `this.fieldName` inside the class.

**Constraints and Limitations:**
- Default values: `0` for numeric types, `false` for `boolean`, `null` for references.
- Stored on the heap as part of the object.
- Lifetime tied to the object’s lifetime; eligible for garbage collection when the object is unreachable.

**Annotated Code Examples:**

```java
// Example 5: Instance variables
public class Bicycle {
    // Instance variables (each Bicycle object gets its own copies)
    private int cadence;   // default 0
    private int speed;     // default 0
    private int gear;      // default 0
    
    // Constructor initializes instance variables
    public Bicycle(int startCadence, int startSpeed, int startGear) {
        this.cadence = startCadence;
        this.speed = startSpeed;
        this.gear = startGear;
    }
    
    public void printStates() {
        System.out.println("cadence:" + cadence + " speed:" + speed + " gear:" + gear);
    }
    
    public static void main(String[] args) {
        Bicycle bike1 = new Bicycle(10, 0, 1);
        Bicycle bike2 = new Bicycle(20, 5, 3);
        
        bike1.printStates();  // bike1's own values
        bike2.printStates();  // bike2's own values
    }
}
```

**Expected Output:**
```
cadence:10 speed:0 gear:1
cadence:20 speed:5 gear:3
```

**Why This Output:** Each `Bicycle` instance has its own `cadence`, `speed`, and `gear` fields. The constructor uses `this` to distinguish parameters from fields. Modifying one bike’s state does not affect the other.

**Real-World Case:** A `BankAccount` class has instance variables `balance`, `accountNumber`, and `owner`; each account object maintains its own values.

---

##### Static Variables (Class Variables)

**Definitions:**
- **Core Definition:** Static variables are fields declared with the `static` modifier; there is exactly one copy shared by all instances of the class.
- **Technical Definition:** A class variable is a field declared with the `static` keyword, created when the class is initialized and shared across all instances.
- **Beginner Explanation:** Information that belongs to the class itself, not to any individual object—like a shared whiteboard that all objects can read and write.

**Purposes:**
- **To** store data common to all instances of a class.
- **To** implement counters that track how many objects have been created.
- **To** define constants shared across the class.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
static type variableName = initializer;
```

**Syntax Rules:**
- Accessed via `ClassName.variableName` (preferred) or `objectName.variableName`.
- Initialized when the class is first loaded.
- Automatically initialized to default values if no initializer.

**Constraints and Limitations:**
- One copy per class, regardless of instance count.
- Stored in the method area / heap (JVM implementation-dependent).
- Lifetime tied to the class; garbage collected only if the class is unloaded.

**Annotated Code Examples:**

```java
// Example 6: Static variables shared across instances
public class Bicycle {
    // Static variable: shared by ALL Bicycle objects
    private static int numberOfBicycles = 0;
    
    private int cadence;
    
    public Bicycle(int startCadence) {
        this.cadence = startCadence;
        numberOfBicycles++;  // Increment shared counter
    }
    
    public static int getNumberOfBicycles() {
        return numberOfBicycles;
    }
    
    public static void main(String[] args) {
        System.out.println("Before: " + Bicycle.getNumberOfBicycles());
        
        Bicycle bike1 = new Bicycle(10);
        Bicycle bike2 = new Bicycle(20);
        Bicycle bike3 = new Bicycle(30);
        
        System.out.println("After creating 3 bikes: " + Bicycle.getNumberOfBicycles());
    }
}
```

**Expected Output:**
```
Before: 0
After creating 3 bikes: 3
```

**Why This Output:** `numberOfBicycles` is a single shared counter. Each constructor increments it. The static method `getNumberOfBicycles()` accesses the class variable without needing an instance.

**Real-World Case:** A `DatabaseConnection` class might have a static `connectionCount` field tracking active connections across the entire application.

---

##### Parameters / Arguments

**Definitions:**
- **Core Definition:** Parameters are variables that receive values passed to methods or constructors.
- **Technical Definition:** Parameters are formal variables declared in a method or constructor signature; arguments are the actual values supplied at the call site.
- **Beginner Explanation:** Information you hand to a method so it knows what to work on.

**Purposes:**
- **To** pass data into methods for processing.
- **To** allow methods to operate on different inputs.
- **To** enable constructors to initialize object state.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
returnType methodName(type param1, type param2) { ... }
```

**Syntax Rules:**
- Parameters are declared in the method signature, separated by commas.
- Each parameter has a type and name.
- The number and types of arguments must match the parameters at the call site.

**Constraints and Limitations:**
- Parameters are local to the method; modifications do not affect the caller’s variable (for primitives).
- For reference types, the reference is passed by value; the object’s state can be modified but the reference itself cannot be reassigned to affect the caller.

**Annotated Code Examples:**

```java
// Example 7: Method parameters
public class ParameterDemo {
    // Method with two parameters
    public static int add(int a, int b) {
        return a + b;  // a and b are local to this method
    }
    
    // Method with object parameter
    public static void modifyArray(int[] arr) {
        arr[0] = 999;  // Modifies the object's state
    }
    
    public static void main(String[] args) {
        int result = add(5, 3);  // 5 and 3 are arguments
        System.out.println("Sum: " + result);
        
        int[] numbers = {1, 2, 3};
        System.out.println("Before: " + numbers[0]);
        modifyArray(numbers);  // Passes the reference
        System.out.println("After: " + numbers[0]);
    }
}
```

**Expected Output:**
```
Sum: 8
Before: 1
After: 999
```

**Why This Output:** `add(5, 3)` passes values 5 and 3 into parameters `a` and `b`. `modifyArray` receives a copy of the reference `numbers`; both refer to the same array object, so modifying `arr[0]` changes `numbers[0]`.

**Real-World Case:** A `calculateInterest(double principal, double rate, int years)` method receives three parameters to compute compound interest.

---

### 3. Scope and Lifetime

#### Definitions

**Core Definition:** Scope determines where a variable’s name is visible; lifetime determines how long the variable’s storage persists.

**Technical Definition:** The scope of a declaration is the region of the program within which the entity declared by the declaration can be referred to using a simple name.

**Beginner Explanation:** Scope is “where can I use this variable’s name?” Lifetime is “how long does the variable exist in memory?”

#### Sub-features

##### Method Scope vs. Class Scope

**Definitions:**
- **Core Definition:** Method scope means the variable is visible only within the method. Class scope means the variable is visible throughout the class.
- **Technical Definition:** Class-scope variables (fields) are accessible by all methods of the class; method-scope variables (locals) are accessible only within the declaring method.
- **Beginner Explanation:** Class-scope variables are like a house’s shared rooms; method-scope variables are like items you use only while in the kitchen.

**Purposes:**
- **To** encapsulate state appropriately—fields for persistent state, locals for temporary values.
- **To** prevent unintended interference between methods.
- **To** control access granularity.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
class MyClass {
    type classScopedVariable;              // Class scope
    
    void myMethod() {
        type methodScopedVariable;         // Method scope
    }
}
```

**Syntax Rules:**
- Fields are declared directly in the class body.
- Locals are declared inside method bodies or blocks.
- Fields may be accessed by any method; locals only by their declaring method.

**Constraints and Limitations:**
- Method-scoped variables cannot be accessed by other methods.
- Class-scoped variables exist for the lifetime of the object (instance) or class (static).

**Annotated Code Examples:**

```java
// Example 8: Class scope vs. method scope
public class ScopeDemo {
    private int classScoped = 100;  // Class scope: accessible everywhere in this class
    
    public void methodA() {
        int methodScoped = 200;     // Method scope: only in methodA
        System.out.println("methodA sees: " + classScoped + ", " + methodScoped);
    }
    
    public void methodB() {
        System.out.println("methodB sees: " + classScoped);
        // System.out.println(methodScoped);  // ERROR: not in scope
    }
    
    public static void main(String[] args) {
        ScopeDemo obj = new ScopeDemo();
        obj.methodA();
        obj.methodB();
    }
}
```

**Expected Output:**
```
methodA sees: 100, 200
methodB sees: 100
```

**Why This Output:** `classScoped` is visible in both methods. `methodScoped` is only visible inside `methodA`. `methodB` cannot access `methodScoped`.

**Real-World Case:** A `ShoppingCart` class has a field `totalPrice` (class scope) and a local `itemSubtotal` inside `addItem()` (method scope).

---

##### Block Scope

**Definitions:**
- **Core Definition:** Block scope restricts variable visibility to the enclosing `{ }` block.
- **Technical Definition:** The scope of a local variable declaration in a block is the rest of the block in which the declaration appears, starting with its own initializer and including any further declarators to the right.
- **Beginner Explanation:** Variables declared inside `{ }` are only known inside those braces.

**Purposes:**
- **To** limit variable visibility to where it is needed.
- **To** enable reuse of variable names in different blocks.
- **To** improve code clarity and reduce naming conflicts.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
{
    type variableName = initializer;
    // variableName is visible only here
}
// variableName is not visible here
```

**Syntax Rules:**
- A block is denoted by `{ }`.
- The scope extends from the declaration to the end of the block.
- Nested blocks create nested scopes.

**Constraints and Limitations:**
- Cannot redeclare a variable with the same name in the same block.
- Variables in inner blocks shadow variables with the same name in outer blocks.

**Annotated Code Examples:**

```java
// Example 9: Block scope
public class BlockScopeDemo {
    public static void main(String[] args) {
        int x = 10;  // Visible in main's block
        
        {
            int y = 20;  // Visible only in this inner block
            System.out.println("Inside block: x=" + x + ", y=" + y);
            
            {
                int z = 30;  // Nested block
                System.out.println("Nested: x=" + x + ", y=" + y + ", z=" + z);
            }
            // z is no longer visible
        }
        
        // y is no longer visible
        System.out.println("Outside block: x=" + x);
    }
}
```

**Expected Output:**
```
Inside block: x=10, y=20
Nested: x=10, y=20, z=30
Outside block: x=10
```

**Why This Output:** `x` is visible throughout `main`. `y` is visible only in the first inner block. `z` is visible only in the nested block. After each block closes, its variables go out of scope.

**Real-World Case:** In a method processing form input, an `if` block declares a local `String trimmedInput` that is discarded after validation.

---

##### Shadowing Variables

**Definitions:**
- **Core Definition:** Shadowing occurs when a variable declared in an inner scope hides a variable of the same name in an outer scope.
- **Technical Definition:** A declaration *d* of a variable named *n* shadows a declaration of a variable named *n* throughout the scope of *d*, unless the shadowed declaration is itself shadowed.
- **Beginner Explanation:** When you reuse a name inside a smaller box, it temporarily hides the bigger box’s variable with that name.

**Purposes:**
- **To** allow method parameters to have the same names as fields for readability.
- **To** prevent accidental modification of outer variables.
- **To** enable concise naming in localized contexts.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
class MyClass {
    int value;  // Field
    
    void setValue(int value) {  // Parameter shadows field
        this.value = value;      // Use 'this' to access the field
    }
}
```

**Syntax Rules:**
- Shadowing is allowed (with some restrictions on local variable shadowing other locals in enclosing blocks).
- Use `this.fieldName` to access a shadowed instance field.
- Use `ClassName.staticField` to access a shadowed static field.

**Constraints and Limitations:**
- A local variable cannot shadow another local variable in the same method (compile error).
- Parameters can shadow fields.
- Shadowing can lead to bugs if `this` is forgotten.

**Annotated Code Examples:**

```java
// Example 10: Shadowing instance variables with parameters
public class ShadowDemo {
    private int number = 100;  // Instance variable
    
    public void setNumber(int number) {  // Parameter shadows field
        System.out.println("Parameter: " + number);       // Refers to parameter
        System.out.println("Field before: " + this.number); // Refers to field
        this.number = number;  // Assigns parameter to field
        System.out.println("Field after: " + this.number);
    }
    
    public static void main(String[] args) {
        ShadowDemo obj = new ShadowDemo();
        obj.setNumber(42);
    }
}
```

**Expected Output:**
```
Parameter: 42
Field before: 100
Field after: 42
```

**Why This Output:** The parameter `number` shadows the field `number` inside `setNumber`. `number` refers to the parameter; `this.number` refers to the field. The assignment `this.number = number` copies the parameter’s value into the field.

**Real-World Case:** A `Person` class with a field `name` and constructor parameter `String name` uses `this.name = name` to avoid naming confusion.

---

##### Variable Lifetime

**Definitions:**
- **Core Definition:** Lifetime describes how long a variable’s storage exists in memory.
- **Technical Definition:** Local variables live on the stack and are removed when their block/method exits; instance variables live on the heap as long as their object is reachable; static variables live as long as the class is loaded.
- **Beginner Explanation:** Local variables vanish when the method finishes; object variables vanish when the object is no longer used; static variables last until the program ends.

**Purposes:**
- **To** understand memory usage and garbage collection eligibility.
- **To** reason about object state persistence.
- **To** avoid memory leaks by releasing references.

**Syntax Structures and Rules:**

No special syntax; lifetime is determined by declaration location and usage.

**Constraints and Limitations:**
- Stack variables are automatically reclaimed when the method returns.
- Heap objects become eligible for GC when no references point to them.
- Static variables persist for the JVM’s lifetime (unless the class is unloaded).

**Annotated Code Examples:**

```java
// Example 11: Variable lifetime demonstration
public class LifetimeDemo {
    private static int staticCounter = 0;  // Class lifetime
    private int instanceCounter = 0;       // Object lifetime
    
    public LifetimeDemo() {
        instanceCounter++;  // Each object gets its own copy
        staticCounter++;    // Shared across all objects
    }
    
    public void localLifetime() {
        int local = 0;  // Created on each method call, destroyed on return
        local++;
        System.out.println("Local value: " + local);
    }
    
    public static void main(String[] args) {
        LifetimeDemo obj1 = new LifetimeDemo();  // staticCounter=1, obj1.instance=1
        LifetimeDemo obj2 = new LifetimeDemo();  // staticCounter=2, obj2.instance=1
        
        obj1.localLifetime();  // local=1
        obj1.localLifetime();  // local=1 again (new variable each call)
        
        System.out.println("Static counter: " + staticCounter);
        System.out.println("Obj1 instance: " + obj1.instanceCounter);
        System.out.println("Obj2 instance: " + obj2.instanceCounter);
    }
}
```

**Expected Output:**
```
Local value: 1
Local value: 1
Static counter: 2
Obj1 instance: 1
Obj2 instance: 1
```

**Why This Output:** The local variable `local` is recreated with value 0 on each call, then incremented to 1. `staticCounter` accumulates across both constructor calls. Each object’s `instanceCounter` is incremented once in its own constructor.

**Real-World Case:** A method that processes a file uses a local `BufferedReader` that is eligible for garbage collection when the method returns; a `static` logger instance persists for the application’s lifetime.

---

### 4. Constants

#### Definitions

**Core Definition:** Constants are variables declared with `final` that cannot be reassigned after initialization.

**Technical Definition:** A `final` variable may only be assigned once. It is a compile-time error if a `final` variable is assigned unless it is definitely unassigned immediately prior to the assignment. A *constant variable* is a `final` variable of primitive type or type `String` initialized with a constant expression.

**Beginner-Friendly Explanation:** A constant is a variable whose value is locked in place after you set it—like a permanent marker label that cannot be changed.

#### Sub-features

##### Declaring Constants with `final`

**Definitions:**
- **Core Definition:** The `final` keyword prevents reassignment of a variable.
- **Technical Definition:** A variable can be declared `final`; once assigned, it always contains the same value.
- **Beginner Explanation:** You promise Java that you will never change this variable’s value after setting it.

**Purposes:**
- **To** document that a value should never change.
- **To** enable compiler optimizations (inlining for constant variables).
- **To** prevent programming errors from accidental modification.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
final type variableName = initializer;   // Constant variable (if constant expression)
final type variableName;                 // Blank final
```

**Syntax Rules:**
- `final` is a modifier placed before the type.
- For local variables, `final` can be applied to any local variable.
- For fields, `final` can be applied to instance or static fields.
- The variable must be assigned exactly once.

**Constraints and Limitations:**
- A `final` reference variable cannot be reassigned to refer to a different object, but the object’s internal state can be modified.
- A `final` array reference cannot be reassigned, but elements can be modified.
- Blank finals must be assigned before use (definite assignment).

**Annotated Code Examples:**

```java
// Example 12: Final variables and mutable state
public class FinalDemo {
    // Constant variable: final + primitive + constant expression
    public static final double PI = 3.14159;
    
    // Final reference: object state is mutable
    public static final StringBuilder MESSAGE = new StringBuilder("Hello");
    
    public static void main(String[] args) {
        // PI = 3.14;  // ERROR: cannot assign a value to final variable PI
        
        System.out.println("PI: " + PI);
        
        // MESSAGE = new StringBuilder("World");  // ERROR: cannot reassign reference
        MESSAGE.append(" World");  // OK: modifying object state
        System.out.println("MESSAGE: " + MESSAGE);
        
        // Final local variable
        final int MAX = 100;
        // MAX = 200;  // ERROR
        System.out.println("MAX: " + MAX);
    }
}
```

**Expected Output:**
```
PI: 3.14159
MESSAGE: Hello World
MAX: 100
```

**Why This Output:** `PI` is a constant variable and cannot be reassigned. `MESSAGE` is a `final` reference; the reference cannot change, but the `StringBuilder` object it points to can be modified. `MAX` is a local `final` variable.

**Real-World Case:** A `MathUtils` class defines `public static final double E = 2.71828;` as a mathematical constant.

---

##### Compile-Time Constants vs. Runtime Constants

**Definitions:**
- **Core Definition:** Compile-time constants have values known at compile time; runtime constants are `final` but initialized at runtime.
- **Technical Definition:** A *constant variable* is a `final` variable of primitive type or type `String` initialized with a constant expression. Other `final` variables are runtime constants.
- **Beginner Explanation:** Some constants are so simple that Java can bake them into the code at compile time; others can only be set when the program runs.

**Purposes:**
- **To** enable compile-time optimization and inlining.
- **To** support `switch` statements with constant labels.
- **To** distinguish between values fixed at development time and those determined at runtime.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
static final int COMPILE_TIME = 10;                    // Compile-time constant
static final int RUNTIME = computeValue();             // Runtime constant
```

**Syntax Rules:**
- A compile-time constant expression consists of literals, `final` variables initialized with constant expressions, and certain operators.
- The type must be primitive or `String`.
- Runtime constants are `final` variables whose initializer is not a constant expression.

**Constraints and Limitations:**
- Compile-time constants are inlined at usage sites; changing them requires recompilation of dependent code.
- `switch` case labels must be compile-time constants.

**Annotated Code Examples:**

```java
// Example 13: Compile-time vs. runtime constants
public class ConstantTypesDemo {
    // Compile-time constant: value known at compile time
    public static final int MAX_USERS = 100;
    
    // Runtime constant: value determined at runtime
    public static final long START_TIME = System.currentTimeMillis();
    
    // Runtime constant: initialized via method call
    public static final String VERSION = getVersion();
    
    private static String getVersion() {
        return "1.0.0";
    }
    
    public static void main(String[] args) {
        // Compile-time constant can be used in switch
        int choice = 1;
        switch (choice) {
            case MAX_USERS:  // Valid: MAX_USERS is compile-time constant
                System.out.println("Max users selected");
                break;
            default:
                System.out.println("Other choice");
        }
        
        System.out.println("Start time: " + START_TIME);
        System.out.println("Version: " + VERSION);
    }
}
```

**Expected Output (example):**
```
Other choice
Start time: 1699999999999
Version: 1.0.0
```

**Why This Output:** `MAX_USERS` is a compile-time constant, so it can be used as a `case` label. `START_TIME` and `VERSION` are runtime constants; their values are determined when the class is initialized.

**Real-World Case:** A `Config` class might have `public static final String API_URL = System.getenv("API_URL");` as a runtime constant.

---

##### Blank Finals

**Definitions:**
- **Core Definition:** A blank final is a `final` variable without an initializer, assigned exactly once later.
- **Technical Definition:** A *blank final* is a `final` variable whose declaration lacks an initializer. A blank final instance variable must be definitely assigned at the end of every constructor.
- **Beginner Explanation:** You declare a constant but don’t give it a value yet—you must set it exactly once, usually in the constructor.

**Purposes:**
- **To** allow `final` fields to be initialized based on constructor parameters.
- **To** support dependency injection and configuration.
- **To** enforce immutability while allowing flexible initialization.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
final type variableName;   // Blank final declaration
// In constructor:
variableName = value;      // Single assignment
```

**Syntax Rules:**
- Blank finals must be assigned exactly once before use.
- Instance blank finals must be assigned in every constructor.
- Static blank finals must be assigned in a static initializer.

**Constraints and Limitations:**
- Cannot be read before assignment.
- Cannot be assigned more than once.
- Must be assigned on all code paths through the constructor.

**Annotated Code Examples:**

```java
// Example 14: Blank final instance variable
public class Employee {
    private final String name;      // Blank final
    private final int id;           // Blank final
    private static final int BASE_ID;  // Static blank final
    
    static {
        BASE_ID = 1000;  // Static blank final assigned in static initializer
    }
    
    public Employee(String name, int idOffset) {
        this.name = name;                    // Assign blank final
        this.id = BASE_ID + idOffset;        // Assign blank final
    }
    
    public static void main(String[] args) {
        Employee emp = new Employee("Alice", 42);
        System.out.println("Name: " + emp.name);
        System.out.println("ID: " + emp.id);
    }
}
```

**Expected Output:**
```
Name: Alice
ID: 1042
```

**Why This Output:** `name` and `id` are blank finals assigned in the constructor. `BASE_ID` is a static blank final assigned in a static initializer. Each blank final receives exactly one value.

**Real-World Case:** An immutable `Point` class with `final int x, y` assigned in a constructor based on parameters.

---

### 5. Modern Java Additions

#### Local Variable Type Inference (`var`)

**Definitions:**
- **Core Definition:** The `var` keyword allows local variable types to be inferred from the initializer.
- **Technical Definition:** In JDK 10 and later, `var` can be used for local variable declarations with initializers, enhanced `for` loop indices, traditional `for` loop indices, and try-with-resources variables.
- **Beginner Explanation:** Instead of writing the type twice, you can say `var` and let Java figure it out from the value you assign.

**Purposes:**
- **To** reduce boilerplate in local variable declarations.
- **To** improve readability when the type is obvious from the initializer.
- **To** make code more concise without sacrificing static typing.

**Syntax Structures and Rules:**

**Complete General Syntax:**
```
var variableName = initializer;
```

**Syntax Rules:**
- `var` can only be used for local variables with initializers.
- `var` cannot be used for fields, method parameters (except lambda), or return types.
- The initializer must be present; `var x;` is illegal.
- The inferred type is the static type of the initializer.

**Constraints and Limitations:**
- Cannot use `var` without an initializer.
- Cannot use `var` with `null` as the initializer (ambiguous type).
- Cannot mix `var` and explicit types in lambda parameters.
- `var` is a reserved type name, not a keyword; existing code using `var` as a variable name still works.

**Annotated Code Examples:**

```java
// Example 15: Local variable type inference with var
import java.util.ArrayList;
import java.util.List;

public class VarDemo {
    public static void main(String[] args) {
        // Without var: redundant
        ArrayList<String> list1 = new ArrayList<String>();
        
        // With var: type inferred
        var list2 = new ArrayList<String>();  // infers ArrayList<String>
        
        // Enhanced for loop
        var numbers = List.of(1, 2, 3, 4, 5);
        for (var num : numbers) {             // infers Integer
            System.out.print(num + " ");
        }
        System.out.println();
        
        // Traditional for loop
        for (var i = 0; i < 3; i++) {         // infers int
            System.out.println("i = " + i);
        }
        
        // Try-with-resources
        // var reader = new java.io.FileReader("file.txt");  // infers FileReader
        
        // var with lambda (JDK 11+)
        // BiFunction<Integer, Integer, Integer> f = (var a, var b) -> a + b;
    }
}
```

**Expected Output:**
```
1 2 3 4 5 
i = 0
i = 1
i = 2
```

**Why This Output:** `var list2` infers `ArrayList<String>` from the `new ArrayList<String>()` expression. `var num` infers `Integer` from the list’s element type. `var i` infers `int` from the literal `0`.

**Real-World Case:** Processing a stream of data: `var result = data.stream().filter(x -> x > 0).collect(Collectors.toList());` — the type is clear from the context and `var` reduces verbosity.

---

## References

- Java Language Specification, Chapter 4: Types, Values, and Variables - https://docs.oracle.com/javase/specs/jls/se23/html/jls-4.html
- Java Language Specification, Section 4.12.4: final Variables - https://docs.oracle.com/javase/specs/jls/se6/html/typesValues.html#4.12.4
- Java Tutorials: Variables - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/variables.html
- Dev.java: Creating Primitive Type Variables in Your Programs - https://dev.java/learn/language-basics/primitive-types/
- Java Language Updates: Local Variable Type Inference - https://docs.oracle.com/en/java/javase/26/language/local-variable-type-inference.html
- Java Language Specification, Section 6.4: Shadowing and Obscuring - https://docs.oracle.com/javase/specs/jls/se23/html/jls-6.html#jls-6.4
- OpenJDK: Blank Final Assignment (JLS 8.3.1.2) - https://stackoverflow.com/revisions/e422c68f-5075-4456-b5a5-cb81b8626be1/view-source
- OpenJDK: Lazy Finals Proposal - https://cr.openjdk.org/~jrose/draft/lazy-final.md