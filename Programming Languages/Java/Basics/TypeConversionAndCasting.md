# Java Type Conversion and Casting: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:**
Java type conversion (also called casting) is the process of converting a value from one data type to another within the Java type system. Because Java is a statically typed language, every expression has a known type at compile time, and conversions between types follow strict, specified rules.

**Technical Definition:**
In the Java Language Specification (JLS), type conversions are formally divided into 13 categories, including widening primitive conversion, narrowing primitive conversion, widening reference conversion, narrowing reference conversion, boxing conversion, unboxing conversion, and string conversion. A conversion may occur implicitly (performed automatically by the compiler in assignment, method invocation, or casting contexts) or explicitly (forced by the programmer using the cast operator). Conversions are classified as _exact_ if they preserve all information, and _inexact_ if they lose magnitude, precision, range, or sign.

**Beginner-Friendly Explanation:**
Imagine you have different-sized containers for storing numbers: tiny boxes, medium boxes, and large boxes. Java type conversion is like moving items between these containers. Moving from a small box to a large box is easy and safe—nothing gets lost. Moving from a large box to a small box is riskier—things might not fit, so you must explicitly say “I know what I’m doing” by using a cast. Similarly, Java has two parallel worlds: primitive types (fast, simple values) and object types (richer, but heavier). Converting between these worlds—and between objects in an inheritance hierarchy—follows its own special rules.

### Key Characteristics

- **Static type checking:** Java verifies type safety at compile time. Unsafe assignments without explicit casts are rejected.
- **Automatic vs. explicit:** Widening conversions happen automatically; narrowing and reference downcasts require explicit cast operators.
- **Runtime safety checks:** Reference downcasts are checked at runtime and throw `ClassCastException` if incompatible; unboxing `null` throws `NullPointerException`.
- **Numeric promotion in expressions:** Smaller types (`byte`, `short`, `char`) are automatically promoted to `int` before arithmetic operations.
- **Wrapper class interoperability:** Java’s compiler automatically boxes primitives into wrapper objects and unboxes them back when needed.
- **Deprecated constructors:** Wrapper class constructors (e.g., `new Integer(5)`) have been deprecated since Java 9; static factory methods like `valueOf()` are preferred.

### Prerequisites

To fully understand type conversion and casting in Java, you should be familiar with:

- **Primitive data types** in Java (`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`) and their sizes/ranges.
- **Reference types and inheritance:** classes, subclasses, and the `extends` keyword (for upcasting/downcasting).
- **The `Object` class** as the root of the Java class hierarchy.
- **Basic operators and expressions** (arithmetic operators, assignment operators).

### Related Programming Areas

- **Generics:** Generics in Java only work with reference types, which is why autoboxing is essential for storing primitives in collections like `ArrayList<Integer>`.
- **Collections Framework:** Converting primitives to wrapper objects enables storage in collections, but introduces performance overhead and `NullPointerException` risks.
- **Reflection:** The `Class` class and reflection APIs frequently require casts when retrieving objects dynamically.
- **I/O and Serialization:** String conversion (`parseInt`, `String.valueOf`) is essential when reading data from files, user input, or network protocols.
- **Pattern Matching (Modern Java):** Recent Java versions (preview features) allow primitive types in `instanceof` and `switch` patterns, requiring exact testing conversions.

## Primitive Type Conversions

**Core Definition:**
Primitive type conversion is the process of converting a value from one primitive data type to another. Java defines 19 widening primitive conversions and 22 narrowing primitive conversions across its 8 primitive types.

**Technical Definition:**
A _widening primitive conversion_ converts a value to a type with a strictly larger range of representable values. It is specified as never throwing a runtime exception, though it may lose precision in certain cases (e.g., `int` to `float`). A _narrowing primitive conversion_ converts a value to a type with a smaller or different range. It may lose magnitude, precision, range, or sign, and is performed by discarding the high-order bits of the source value for integral types.

**Beginner-Friendly Explanation:**
Widening is like pouring a small cup of water into a big bucket—nothing spills. Narrowing is like pouring that big bucket into a small cup—water will spill unless you’re very careful. Java lets you do widening without asking, but for narrowing, you must say “I accept the risk” by writing a cast.

### Widening Conversion

**Definitions:**

- **Core Definition:** Automatic conversion from a smaller primitive type to a larger primitive type that preserves the value’s magnitude.
- **Technical Definition:** One of 19 specified conversions where the target type has a strictly larger range. Integral widening sign-extends signed values and zero-extends `char` values.
- **Beginner-Friendly Explanation:** Java does this for you automatically because it is safe—the new container is big enough to hold everything from the old one.

**Purposes:**

- To safely store smaller values in larger variables without explicit syntax.
- To enable mixed-type arithmetic expressions to evaluate without errors.
- To allow polymorphic-like behavior for numeric types in method calls.
- To simplify code by removing unnecessary casts where no data loss is possible.

**Syntax Structures and Rules:**

**General Syntax:**

```java
targetType variableName = sourceValue;  // Implicit widening
```

**Component Breakdown:**

- `targetType`: The larger primitive type (e.g., `long`, `double`).
- `variableName`: The identifier for the new variable.
- `sourceValue`: An expression of a smaller primitive type.

**Widening Conversion Hierarchy:**

```
byte → short → int → long → float → double
char  → int → long → float → double
```

**Syntax Rules:**

- Widening conversions are **implicit** in assignment, method invocation, and casting contexts.
- No cast operator is required.
- Widening never results in a runtime exception.

**Constraints and Limitations:**

- `int` to `float` and `long` to `float`/`double` may lose **precision** (least significant bits) even though they are widening conversions. Example: `1234567890` as `int` becomes `1.23456794E9` as `float`, losing the exact value.
- `boolean` cannot be converted to or from any other primitive type.

**Multiple Annotated Complete Code Examples:**

**Example 1: Widening in Assignment**

```java
public class WideningDemo {
    public static void main(String[] args) {
        // byte to int: safe, no cast needed
        byte smallValue = 42;
        int widenedInt = smallValue;  // Implicit widening
        System.out.println("Byte 42 as int: " + widenedInt);  // 42

        // int to double: safe, no cast needed
        int integerValue = 100;
        double widenedDouble = integerValue;  // Implicit widening
        System.out.println("Int 100 as double: " + widenedDouble);  // 100.0

        // char to int: character promoted to Unicode code point
        char letter = 'A';
        int asciiValue = letter;  // Implicit widening
        System.out.println("Char 'A' as int: " + asciiValue);  // 65
    }
}
```

**Expected Output:**

```
Byte 42 as int: 42
Int 100 as double: 100.0
Char 'A' as int: 65
```

**Why This Output:**

- `42` (byte) fits exactly into an `int`; the value is unchanged.
- `100` (int) becomes `100.0` (double); the decimal point is added because double represents fractional values.
- `'A'` has Unicode code point 65; widening to `int` exposes this numeric value.

**Example 2: Widening with Precision Loss**

```java
public class PrecisionLossDemo {
    public static void main(String[] args) {
        int bigNumber = 1234567890;
        float approximate = bigNumber;  // Widening int→float, but loses precision
        System.out.println("Original int: " + bigNumber);
        System.out.println("As float: " + approximate);

        // Round-trip check (illustrative only; not a reliable safety test)
        int roundTrip = (int) approximate;
        System.out.println("Cast back to int: " + roundTrip);
        System.out.println("Are they equal? " + (bigNumber == roundTrip));
    }
}
```

**Expected Output:**

```
Original int: 1234567890
As float: 1.23456794E9
Cast back to int: 1234567936
Are they equal? false
```

**Why This Output:**

- `float` has 24 bits of mantissa precision; `1234567890` requires more precision than `float` can represent.
- The value is rounded to the nearest representable `float`, losing the least significant digits.
- Casting back does not recover the original value, demonstrating that widening from `int` to `float` can be inexact despite being called “widening”.

**Real-World Cases:**

1. **Scientific computation:** Storing integer measurements as `double` for fractional calculations.
2. **Banking applications:** Using `long` for monetary cents and widening to `double` for display purposes (though `BigDecimal` is preferred for money).
3. **Character processing:** Converting `char` to `int` to compute offsets, cipher shifts, or Unicode manipulations.

**References:**

- Java Language Specification, §5.1.2 Widening Primitive Conversion - https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html
- Oracle Java Tutorials: Primitive Data Types - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html

### Narrowing Conversion and Explicit Casts

**Definitions:**

- **Core Definition:** Manual conversion from a larger primitive type to a smaller one, requiring a cast operator and potentially losing data.
- **Technical Definition:** One of 22 specified conversions where the target type may have a smaller range. For integral types, the conversion discards all but the _n_ lowest-order bits of the source value, where _n_ is the target type’s bit width.
- **Beginner-Friendly Explanation:** You’re forcing a big value into a small box. Java makes you write `(targetType)` to show you understand that some data might be lost or the value might become strange.

**Purposes:**

- To convert floating-point results to integers when fractional parts are no longer needed.
- To store large-range values in compact memory when the programmer knows the value fits.
- To perform bitwise operations that require specific integer widths.
- To truncate values intentionally for algorithms like hashing or checksums.

**Syntax Structures and Rules:**

**General Syntax:**

```java
targetType variableName = (targetType) sourceValue;
```

**Component Breakdown:**

- `(targetType)`: The cast operator, consisting of the target type in parentheses.
- `sourceValue`: An expression of a larger or different primitive type.

**Syntax Rules:**

- The cast operator `(type)` must immediately precede the value to be converted.
- Narrowing is allowed in **assignment** and **casting** contexts but not in method invocation contexts without explicit cast.
- If the source value is a **constant expression** (compile-time constant) that fits in the target type, the compiler may allow narrowing without a cast (e.g., `byte b = 100;`).

**Constraints and Limitations:**

- **Information loss:** Narrowing can lose magnitude, precision, range, and sign.
- **Special float-to-int behavior:** `NaN` becomes `0`; values too large become `Integer.MAX_VALUE` or `Integer.MIN_VALUE`; rounding is toward zero.
- **Integral overflow:** Values larger than the target type “wrap around” based on two’s complement representation.
- Narrowing is **never** automatic in assignment of non-constant expressions.

**Multiple Annotated Complete Code Examples:**

**Example 1: Double to Int Narrowing**

```java
public class NarrowingDemo {
    public static void main(String[] args) {
        double price = 99.99;

        // Explicit cast: fractional part is truncated (not rounded)
        int truncated = (int) price;
        System.out.println("Double 99.99 cast to int: " + truncated);  // 99

        // Large double beyond int range: saturates to Integer.MAX_VALUE
        double huge = 1e20;
        int saturated = (int) huge;
        System.out.println("Double 1e20 cast to int: " + saturated);  // 2147483647

        // Negative double: truncation toward zero
        double negative = -42.9;
        int negTruncated = (int) negative;
        System.out.println("Double -42.9 cast to int: " + negTruncated);  // -42
    }
}
```

**Expected Output:**

```
Double 99.99 cast to int: 99
Double 1e20 cast to int: 2147483647
Double -42.9 cast to int: -42
```

**Why This Output:**

- `99.99` truncates toward zero to `99` (the `.99` is discarded, not rounded).
- `1e20` exceeds `int`’s maximum of `2147483647`, so Java saturates to that maximum.
- `-42.9` truncates toward zero to `-42` (not `-43`).

**Example 2: Integral Overflow During Narrowing**

```java
public class OverflowDemo {
    public static void main(String[] args) {
        int bigInt = 300;

        // 300 in binary: 100101100 (9 bits)
        // byte is 8 bits: keeps only lowest 8 bits: 00101100 = 44
        byte overflowByte = (byte) bigInt;
        System.out.println("300 cast to byte: " + overflowByte);  // 44

        // Character value beyond byte range
        char maxChar = Character.MAX_VALUE;  // 65535
        short signedShort = (short) maxChar;
        System.out.println("char 65535 cast to short: " + signedShort);  // -1

        // Constant expression within range: compiler allows without cast
        byte validByte = 100;  // No cast needed; 100 fits in byte
        System.out.println("Valid byte: " + validByte);  // 100
    }
}
```

**Expected Output:**

```
300 cast to byte: 44
char 65535 cast to short: -1
Valid byte: 100
```

**Why This Output:**

- `300` in 32-bit binary: `00000000 00000000 00000001 00101100`. Keeping only the lowest 8 bits (`00101100`) yields `44`.
- `char` 65535 is `11111111 11111111` in 16-bit unsigned. Reinterpreting as signed `short` yields `-1` (two’s complement).
- `100` is a constant expression that fits in `byte`, so the compiler permits narrowing without a cast.

**Real-World Cases:**

1. **Game development:** Truncating floating-point coordinates to integer grid positions.
2. **Network protocols:** Packing values into fixed-width fields (e.g., 16-bit ports, 8-bit flags).
3. **Cryptography:** Explicit bit-width conversions for hash functions and block ciphers.
4. **Graphics programming:** Converting color channel values from `int` to `byte`.

**References:**

- Java Language Specification, §5.1.3 Narrowing Primitive Conversion - https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html
- Java Language Specification, §5.5 Casting Contexts - https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html

### Numeric Promotion in Expressions

**Definitions:**

- **Core Definition:** Automatic widening of smaller numeric types to `int` (or a common larger type) during arithmetic operations.
- **Technical Definition:** Java’s binary numeric promotion rules specify that if either operand is `double`, both become `double`; else if either is `float`, both become `float`; else if either is `long`, both become `long`; otherwise both become `int`. Unary numeric promotion similarly promotes `byte`, `short`, and `char` to `int`.
- **Beginner-Friendly Explanation:** When you do math with a `byte` and an `int`, Java first “upgrades” the `byte` to an `int` so both are the same type. This prevents overflow during the calculation itself.

**Purposes:**

- To prevent intermediate arithmetic overflow when operations involve small types.
- To provide a uniform type for binary operations between mixed numeric types.
- To enable arithmetic on `char` values by promoting them to `int`.
- To ensure consistent behavior across platforms with different native integer widths.

**Syntax Structures and Rules:**

**General Syntax (implicit within expressions):**

```java
resultType result = operand1 operator operand2;  // Promotion applied automatically
```

**Component Breakdown:**

- `operand1`, `operand2`: Numeric expressions of possibly different types.
- `operator`: `+`, `-`, `*`, `/`, `%`, or comparison operators.

**Promotion Rules:**

1. **Unary promotion:** `byte`, `short`, `char` → `int`.
2. **Binary promotion:** If either operand is `double` → both `double`. Else if either is `float` → both `float`. Else if either is `long` → both `long`. Else both `int`.

**Syntax Rules:**

- Promotion is automatic and invisible in source code.
- The result of an arithmetic expression involving any `int` or smaller is at least `int`.
- Assigning a promoted result back to a smaller type requires an explicit cast.

**Constraints and Limitations:**

- You cannot perform arithmetic directly on `boolean`.
- The promotion may cause **compile errors** when you try to assign the result to a smaller variable without a cast.
- Promotion does **not** occur for string concatenation; instead, other operands are converted to strings.

**Multiple Annotated Complete Code Examples:**

**Example 1: Byte Arithmetic Requires Cast to Store**

```java
public class PromotionDemo {
    public static void main(String[] args) {
        byte a = 10;
        byte b = 20;

        // a + b is promoted to int, so result is int
        // byte sum = a + b;  // COMPILE ERROR: int cannot be assigned to byte

        // Correct: explicit cast
        byte sum = (byte)(a + b);
        System.out.println("Byte sum: " + sum);  // 30

        // Mixed types: entire expression promoted to double
        int quantity = 5;
        double unitPrice = 12.50;
        double total = quantity * unitPrice;  // int promoted to double
        System.out.println("Total: " + total);  // 62.5
    }
}
```

**Expected Output:**

```
Byte sum: 30
Total: 62.5
```

**Why This Output:**

- `a + b` is calculated as `int` (10 + 20 = 30), then cast back to `byte`.
- `quantity * unitPrice`: `quantity` is promoted to `5.0` (double), then multiplied by `12.50`, yielding `62.5`.

**Example 2: Character Arithmetic**

```java
public class CharPromotionDemo {
    public static void main(String[] args) {
        char letter = 'A';  // Unicode 65

        // Arithmetic promotes char to int
        int nextCode = letter + 1;  // 65 + 1 = 66
        System.out.println("Next code: " + nextCode);  // 66

        // Cast back to char to get the character
        char nextLetter = (char)(letter + 1);
        System.out.println("Next letter: " + nextLetter);  // B

        // Finding the 5th letter
        char fifthLetter = (char)('A' + 4);
        System.out.println("5th letter: " + fifthLetter);  // E
    }
}
```

**Expected Output:**

```
Next code: 66
Next letter: B
5th letter: E
```

**Why This Output:**

- `letter + 1`: `letter` (char, 65) is promoted to `int`, giving `66`.
- `(char)(letter + 1)`: casting `66` back to `char` yields `'B'`.
- `'A' + 4`: `'A'` is 65, plus 4 gives 69, cast to `char` yields `'E'`.

**Real-World Cases:**

1. **Encryption algorithms:** Caesar cipher, ROT13, and other character-shift ciphers rely on `char` promotion.
2. **Financial calculations:** Mixing `int` counts with `double` prices requires promotion to avoid truncation.
3. **Graphics:** Pixel arithmetic often promotes `byte` channel values to `int` for blending operations.

**References:**

- Java Language Specification, §5.6 Numeric Promotions - https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html
- Oracle Java Tutorials: Operators - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html

## Reference (Object) Type Conversions

**Core Definition:**
Reference type conversion involves moving references between classes in an inheritance hierarchy. Upcasting goes from subclass to superclass; downcasting goes from superclass to subclass.

**Technical Definition:**
A _widening reference conversion_ converts from a subclass type to a superclass (or interface) type and is always safe at compile time. A _narrowing reference conversion_ converts from a superclass type to a subclass type and requires an explicit cast; it is checked at runtime and throws `ClassCastException` if the object is not actually an instance of the target type.

**Beginner-Friendly Explanation:**
Imagine you have a `Dog` object. You can always treat it as an `Animal` because every dog is an animal. That’s upcasting—safe and automatic. But if you have an `Animal` reference, you can’t assume it’s a `Dog` without checking; it might be a `Cat`. You must explicitly cast it, and Java will check at runtime to make sure it really is a dog before letting you proceed.

### Upcasting

**Definitions:**

- **Core Definition:** Converting a subclass reference to a superclass reference automatically.
- **Technical Definition:** A widening reference conversion permitted in assignment, method invocation, and casting contexts without an explicit cast.
- **Beginner-Friendly Explanation:** Treating a specific object as a more general one. You lose access to subclass-specific methods, but the object itself doesn’t change.

**Purposes:**

- To write polymorphic code that works with multiple subclasses.
- To store heterogeneous objects in a common collection type.
- To pass objects to methods that accept a superclass parameter.
- To hide implementation details behind a general interface.

**Syntax Structures and Rules:**

**General Syntax:**

```java
Superclass reference = subclassObject;  // Implicit upcast
```

**Component Breakdown:**

- `Superclass`: The parent class or interface type.
- `subclassObject`: An instance of a class that extends/implements `Superclass`.

**Syntax Rules:**

- Upcasting is **always implicit** and requires no cast.
- Works for both class inheritance and interface implementation.
- The object’s actual type does not change—only the reference type changes.

**Constraints and Limitations:**

- After upcasting, you can only access members defined in the superclass/interface.
- To call subclass-specific methods, you must downcast back.

**Multiple Annotated Complete Code Examples:**

**Example 1: Basic Upcasting**

```java
class Animal {
    void eat() { System.out.println("Animal eats"); }
}

class Dog extends Animal {
    void bark() { System.out.println("Dog barks"); }
}

public class UpcastDemo {
    public static void main(String[] args) {
        Dog myDog = new Dog();

        // Implicit upcast: Dog is an Animal
        Animal animalRef = myDog;

        animalRef.eat();  // Works: eat() is defined in Animal
        // animalRef.bark();  // COMPILE ERROR: Animal has no bark()

        // The actual object is still a Dog
        System.out.println("Is it still a Dog? " + (myDog instanceof Dog));  // true
    }
}
```

**Expected Output:**

```
Animal eats
Is it still a Dog? true
```

**Why This Output:**

- `animalRef.eat()` calls the `eat()` method inherited from `Animal`.
- `bark()` is not accessible through the `Animal` reference because `Animal` doesn’t declare it.
- The object itself remains a `Dog`; upcasting only changes the reference type.

**Real-World Cases:**

1. **Collections:** Storing `ArrayList<Animal>` containing `Dog`, `Cat`, and `Bird` objects.
2. **Event handling:** Listeners registered as `ActionListener` interfaces.
3. **Drawing applications:** A list of `Shape` references pointing to `Circle`, `Rectangle`, and `Triangle` objects.

**References:**

- Java Language Specification, §5.1.5 Widening Reference Conversion - https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html
- Oracle Java Tutorials: Polymorphism - https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html

### Downcasting

**Definitions:**

- **Core Definition:** Converting a superclass reference back to a subclass reference, requiring an explicit cast and runtime type checking.
- **Technical Definition:** A narrowing reference conversion permitted only in casting contexts with an explicit cast operator; the JVM checks at runtime that the object is an instance of the target type, throwing `ClassCastException` otherwise.
- **Beginner-Friendly Explanation:** You have an `Animal` reference and you’re saying “I know this is actually a `Dog`.” Java checks at runtime; if you’re wrong, it throws an error.

**Purposes:**

- To access subclass-specific methods after upcasting.
- To recover the specific type of an object from a heterogeneous collection.
- To implement polymorphic algorithms that branch on runtime type.

**Syntax Structures and Rules:**

**General Syntax:**

```java
Subclass reference = (Subclass) superclassReference;
```

**Component Breakdown:**

- `(Subclass)`: Cast operator specifying the target type.
- `superclassReference`: The reference to be downcast.

**Syntax Rules:**

- Explicit cast is always required.
- Runtime check: if object is not an instance of target type, `ClassCastException` is thrown.
- Use `instanceof` to check before downcasting to avoid exceptions.

**Constraints and Limitations:**

- Downcasting to an unrelated class is a **compile-time error**.
- Even valid downcasts can fail at runtime if the object is a different subclass.
- Downcasting `null` always succeeds (returns `null`).

**Multiple Annotated Complete Code Examples:**

**Example 1: Safe Downcasting with instanceof**

```java
class Animal {
    void eat() { System.out.println("Animal eats"); }
}

class Dog extends Animal {
    void bark() { System.out.println("Dog barks"); }
}

class Cat extends Animal {
    void meow() { System.out.println("Cat meows"); }
}

public class DowncastDemo {
    public static void main(String[] args) {
        Animal[] animals = { new Dog(), new Cat(), new Dog() };

        for (Animal a : animals) {
            if (a instanceof Dog) {
                Dog d = (Dog) a;  // Safe downcast
                d.bark();
            } else if (a instanceof Cat) {
                Cat c = (Cat) a;  // Safe downcast
                c.meow();
            }
        }
    }
}
```

**Expected Output:**

```
Dog barks
Cat meows
Dog barks
```

**Why This Output:**

- The `instanceof` check verifies the actual object type before casting.
- Each downcast succeeds because the check guarantees compatibility.

**Example 2: Unsafe Downcast Throws Exception**

```java
public class UnsafeDowncastDemo {
    public static void main(String[] args) {
        Animal animal = new Animal();

        try {
            Dog d = (Dog) animal;  // Compiles, but throws at runtime
            d.bark();
        } catch (ClassCastException e) {
            System.out.println("Downcast failed: " + e.getMessage());
        }
    }
}

class Animal { }
class Dog extends Animal {
    void bark() { }
}
```

**Expected Output:**

```
Downcast failed: class Animal cannot be cast to class Dog
```

**Why This Output:**

- The `Animal` object is not actually a `Dog`, so the runtime check fails.
- `ClassCastException` is thrown and caught by the `try-catch` block.

**Real-World Cases:**

1. **Event dispatching:** Determining the specific type of a generic event object.
2. **Serialization/deserialization:** Recovering the concrete type of a serialized object.
3. **Plugin architectures:** Loading classes dynamically and casting to known interfaces.

**References:**

- Java Language Specification, §5.5 Casting Contexts - https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html
- Oracle Java Tutorials: Inheritance - https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html

## Object and Primitive Interoperability

**Core Definition:**
Java provides wrapper classes for each primitive type, and the compiler automatically converts between primitives and their wrapper objects through autoboxing and unboxing.

**Technical Definition:**
_Autoboxing_ is the automatic conversion the Java compiler performs from a primitive type to its corresponding wrapper class (e.g., `int` → `Integer`). _Unboxing_ is the reverse (e.g., `Integer` → `int`). The compiler applies these conversions when primitives are assigned to wrapper variables, passed to methods expecting objects, or used in expressions requiring objects.

**Beginner-Friendly Explanation:**
Java has two worlds: the “fast world” of primitive values (like `int`) and the “object world” of classes (like `Integer`). Collections like `ArrayList` only live in the object world. Autoboxing is Java’s way of automatically wrapping a primitive into an object when needed; unboxing unwraps it back.

### Autoboxing

**Definitions:**

- **Core Definition:** Automatic conversion of a primitive value to its corresponding wrapper class object.
- **Technical Definition:** The compiler inserts calls to static factory methods like `Integer.valueOf()` when a primitive is used in an object context.
- **Beginner-Friendly Explanation:** You write `Integer x = 42;` and Java automatically turns `42` into an `Integer` object.

**Purposes:**

- To store primitive values in collections that require objects.
- To pass primitive values to methods expecting wrapper objects.
- To assign primitives to wrapper variables without explicit constructor calls.
- To simplify code by eliminating manual `new Integer(...)` calls.

**Syntax Structures and Rules:**

**General Syntax:**

```java
WrapperClass variable = primitiveValue;  // Autoboxing
```

**Component Breakdown:**

- `WrapperClass`: The wrapper type (`Integer`, `Double`, `Character`, etc.).
- `primitiveValue`: A primitive expression.

**Primitive-to-Wrapper Mapping:**

| Primitive | Wrapper Class |
| --------- | ------------- |
| `byte`    | `Byte`        |
| `short`   | `Short`       |
| `int`     | `Integer`     |
| `long`    | `Long`        |
| `float`   | `Float`       |
| `double`  | `Double`      |
| `char`    | `Character`   |
| `boolean` | `Boolean`     |

**Syntax Rules:**

- Autoboxing occurs in **assignment** and **method invocation** contexts.
- The compiler uses `valueOf()` methods (since Java 5); constructors like `new Integer(5)` are deprecated since Java 9.
- Autoboxing does **not** work for arrays: `int[]` cannot be automatically converted to `Integer[]`.

**Constraints and Limitations:**

- Autoboxing creates **new objects** (or reuses cached ones for small values), introducing memory overhead.
- Cached values: `Integer.valueOf()` caches values from `-128` to `127` by default.
- Using autoboxing in tight loops can cause significant garbage collection pressure.

**Multiple Annotated Complete Code Examples:**

**Example 1: Autoboxing in Collections**

```java
import java.util.ArrayList;
import java.util.List;

public class AutoboxDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();

        // Autoboxing: primitive int 10 becomes Integer
        numbers.add(10);
        numbers.add(20);
        numbers.add(30);

        System.out.println("List: " + numbers);

        // Autoboxing in assignment
        Integer wrapperInt = 42;
        System.out.println("Wrapper Integer: " + wrapperInt);

        // Cached comparison for small values
        Integer a = 100;
        Integer b = 100;
        System.out.println("100 == 100 (cached): " + (a == b));  // true

        Integer c = 1000;
        Integer d = 1000;
        System.out.println("1000 == 1000 (not cached): " + (c == d));  // false
    }
}
```

**Expected Output:**

```
List: [10, 20, 30]
Wrapper Integer: 42
100 == 100 (cached): true
1000 == 1000 (not cached): false
```

**Why This Output:**

- `numbers.add(10)`: `10` is autoboxed to `Integer` before being stored.
- `Integer a = 100; Integer b = 100;`: Both reference the **same cached** `Integer` object for values in `[-128, 127]`, so `==` returns `true`.
- `1000` is outside the cache range, so separate objects are created; `==` compares references and returns `false`.

**Real-World Cases:**

1. **Collections:** Storing counts, scores, or IDs in `List<Integer>`, `Map<String, Integer>`.
2. **Generic methods:** Returning `Optional<Integer>` instead of nullable `int`.
3. **JSON/XML parsing:** Parsing numeric values into wrapper types.

**References:**

- Oracle Java Tutorials: Autoboxing and Unboxing - https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html
- Java Language Specification, §5.1.7 Boxing Conversion - https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html

### Unboxing

**Definitions:**

- **Core Definition:** Automatic conversion of a wrapper object back to its corresponding primitive type.
- **Technical Definition:** The compiler inserts calls to accessor methods like `intValue()`, `doubleValue()` when a wrapper object is used in a primitive context.
- **Beginner-Friendly Explanation:** You have an `Integer` object and need an `int` for math; Java automatically extracts the primitive value.

**Purposes:**

- To use wrapper objects in arithmetic expressions.
- To assign wrapper values to primitive variables.
- To pass wrapper objects to methods expecting primitives.
- To convert collection elements back to primitives for computation.

**Syntax Structures and Rules:**

**General Syntax:**

```java
primitiveType variable = wrapperObject;  // Unboxing
```

**Component Breakdown:**

- `primitiveType`: The primitive type (`int`, `double`, etc.).
- `wrapperObject`: An instance of the corresponding wrapper class.

**Syntax Rules:**

- Unboxing occurs in **assignment** and **method invocation** contexts.
- The compiler calls the appropriate accessor method (e.g., `intValue()`, `doubleValue()`).
- **`NullPointerException` risk:** If the wrapper object is `null`, unboxing throws NPE.

**Constraints and Limitations:**

- Unboxing a `null` wrapper always throws `NullPointerException`.
- Unboxing happens **before** widening: an `Integer` can be unboxed to `int` then widened to `double`.
- Widening does **not** happen before boxing: an `int` cannot be directly autoboxed to `Long`.

**Multiple Annotated Complete Code Examples:**

**Example 1: Unboxing in Arithmetic**

```java
public class UnboxDemo {
    public static void main(String[] args) {
        Integer wrapperValue = 50;

        // Unboxing in arithmetic
        int result = wrapperValue + 10;  // wrapperValue unboxed to int
        System.out.println("Result: " + result);  // 60

        // Unboxing in assignment
        int primitive = wrapperValue;
        System.out.println("Primitive: " + primitive);  // 50

        // Unboxing in comparison
        if (wrapperValue > 40) {
            System.out.println("Greater than 40");
        }
    }
}
```

**Expected Output:**

```
Result: 60
Primitive: 50
Greater than 40
```

**Why This Output:**

- `wrapperValue + 10`: the `Integer` is unboxed to `int` 50, then 10 is added.
- `int primitive = wrapperValue`: direct unboxing.
- `wrapperValue > 40`: unboxed for comparison.

**Example 2: NullPointerException from Unboxing null**

```java
public class NPEUnboxDemo {
    public static void main(String[] args) {
        Integer risky = null;

        try {
            int value = risky;  // Unboxing null throws NPE
            System.out.println(value);
        } catch (NullPointerException e) {
            System.out.println("Unboxing null caused NullPointerException");
        }

        // Safe approach: null check before unboxing
        if (risky != null) {
            int safe = risky;
            System.out.println("Safe: " + safe);
        } else {
            System.out.println("Value was null; using default");
        }
    }
}
```

**Expected Output:**

```
Unboxing null caused NullPointerException
Value was null; using default
```

**Why This Output:**

- `int value = risky;` triggers unboxing on a `null` reference, throwing NPE.
- The second block checks for `null` first, avoiding the exception.

**Real-World Cases:**

1. **Collection processing:** Summing values from a `List<Integer>`.
2. **Optional values:** Handling nullable wrapper types carefully to avoid NPE.
3. **Configuration parsing:** Converting `Integer` values from config maps to primitives.

**References:**

- Oracle Java Tutorials: Autoboxing and Unboxing - https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html
- Java Language Specification, §5.1.8 Unboxing Conversion - https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html

## Textual Conversions

**Core Definition:**
Textual conversion involves converting between strings and other data types—parsing strings into numbers or objects, and converting values into string representations.

**Technical Definition:**
_String parsing_ uses static methods like `Integer.parseInt()`, `Double.parseDouble()` to convert string representations to primitives. _String conversion_ uses `String.valueOf()` or `Object.toString()` to convert values to `String`.

**Beginner-Friendly Explanation:**
When you read “123” from a file or user input, it’s a string, not a number. Parsing turns it into an actual number you can do math with. Going the other way, when you want to display a number or object as text, Java converts it to a string.

### String Parsing

**Definitions:**

- **Core Definition:** Converting a string representation of a value into its actual data type.
- **Technical Definition:** Static methods in wrapper classes (e.g., `Integer.parseInt(String)`) interpret the string according to the type’s format rules and return the corresponding primitive value.
- **Beginner-Friendly Explanation:** “123” is just text until you call `Integer.parseInt("123")`, which gives you the number 123.

**Purposes:**

- To convert user input (always strings) into numeric types.
- To parse configuration values, command-line arguments, and file data.
- To deserialize data from text formats (CSV, JSON).
- To validate and transform textual data into computable form.

**Syntax Structures and Rules:**

**General Syntax:**

```java
primitiveType value = WrapperClass.parsePrimitiveType(string);
```

**Common Parse Methods:**

| Method                         | Returns   | Throws                  |
| ------------------------------ | --------- | ----------------------- |
| `Integer.parseInt(String)`     | `int`     | `NumberFormatException` |
| `Double.parseDouble(String)`   | `double`  | `NumberFormatException` |
| `Long.parseLong(String)`       | `long`    | `NumberFormatException` |
| `Boolean.parseBoolean(String)` | `boolean` | (never throws)          |
| `Byte.parseByte(String)`       | `byte`    | `NumberFormatException` |

**Syntax Rules:**

- The string must contain a valid representation of the target type.
- `NumberFormatException` is thrown for invalid numeric strings.
- `Boolean.parseBoolean()` returns `false` for any string other than `"true"` (case-insensitive).
- Leading/trailing whitespace is **not** allowed in numeric parsing.

**Constraints and Limitations:**

- Radix-sensitive: `Integer.parseInt("FF", 16)` works, but `Integer.parseInt("FF")` throws.
- Overflow: `Integer.parseInt("999999999999")` throws `NumberFormatException`.
- No comma or currency symbols allowed: `"1,000"` fails.

**Multiple Annotated Complete Code Examples:**

**Example 1: Basic String Parsing**

```java
public class ParseDemo {
    public static void main(String[] args) {
        String numberStr = "123";
        String doubleStr = "3.14";
        String boolStr = "true";

        // Parse string to primitive types
        int intValue = Integer.parseInt(numberStr);
        double doubleValue = Double.parseDouble(doubleStr);
        boolean boolValue = Boolean.parseBoolean(boolStr);

        System.out.println("Parsed int: " + intValue);          // 123
        System.out.println("Parsed double: " + doubleValue);    // 3.14
        System.out.println("Parsed boolean: " + boolValue);     // true

        // Parse with radix
        int hexValue = Integer.parseInt("FF", 16);
        System.out.println("Hex FF as int: " + hexValue);       // 255

        // Parse with exception handling
        try {
            int bad = Integer.parseInt("abc");
        } catch (NumberFormatException e) {
            System.out.println("Invalid number format");
        }
    }
}
```

**Expected Output:**

```
Parsed int: 123
Parsed double: 3.14
Parsed boolean: true
Hex FF as int: 255
Invalid number format
```

**Why This Output:**

- Each `parseXxx` method interprets the string according to the type’s rules.
- Radix 16 interprets `"FF"` as hexadecimal 255.
- `"abc"` is not a valid integer, so `NumberFormatException` is caught.

**Real-World Cases:**

1. **Command-line arguments:** Parsing `args[0]` into an integer.
2. **User input validation:** Converting form fields to numbers with error handling.
3. **CSV/TSV parsing:** Converting text columns to numeric types for analysis.

**References:**

- Java API: `Integer.parseInt()` - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Integer.html#parseInt(java.lang.String)
- Java API: `Double.parseDouble()` - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Double.html#parseDouble(java.lang.String)

#### String Conversion

**Definitions:**

- **Core Definition:** Converting any value or object into its string representation.
- **Technical Definition:** `String.valueOf(Object)` calls the object’s `toString()` method, or returns `"null"` for `null` references. For primitives, specialized overloads produce decimal string representations.
- **Beginner-Friendly Explanation:** Turning a number or object into text so you can print it, log it, or send it over a network.

**Purposes:**

- To display values to users (console, GUI, web).
- To concatenate values into log messages or formatted strings.
- To serialize objects to text formats.
- To convert primitives for string-based APIs.

**Syntax Structures and Rules:**

**General Syntax:**

```java
String str = String.valueOf(value);
String str2 = object.toString();
String str3 = "" + value;  // String concatenation
```

**Component Breakdown:**

- `String.valueOf(value)`: Static method accepting any primitive or `Object`.
- `object.toString()`: Instance method on any non-null object.
- `"" + value`: String concatenation operator (uses `String.valueOf` internally).

**Syntax Rules:**

- `String.valueOf(null)` returns `"null"` (no exception).
- `null.toString()` throws `NullPointerException`.
- String concatenation with `+` calls `String.valueOf` on each operand.

**Constraints and Limitations:**

- `toString()` may not provide meaningful output for all objects unless overridden.
- `String.valueOf(char)` is distinct from `String.valueOf(int)`—both are valid.
- Arrays use `Object.toString()` by default (identity hash), not element values; use `Arrays.toString()` instead.

**Multiple Annotated Complete Code Examples:**

**Example 1: String.valueOf and toString**

```java
public class StringConversionDemo {
    public static void main(String[] args) {
        int num = 42;
        double pi = 3.14159;
        Integer wrapper = 100;

        // Using String.valueOf()
        String s1 = String.valueOf(num);
        String s2 = String.valueOf(pi);
        String s3 = String.valueOf(wrapper);
        System.out.println("s1: " + s1);  // "42"
        System.out.println("s2: " + s2);  // "3.14159"
        System.out.println("s3: " + s3);  // "100"

        // Using toString()
        String s4 = wrapper.toString();
        System.out.println("s4: " + s4);  // "100"

        // String concatenation (implicit conversion)
        String s5 = "" + num;
        System.out.println("s5: " + s5);  // "42"

        // Null safety
        Integer nullWrapper = null;
        String nullStr = String.valueOf(nullWrapper);
        System.out.println("nullStr: " + nullStr);  // "null"
    }
}
```

**Expected Output:**

```
s1: 42
s2: 3.14159
s3: 100
s4: 100
s5: 42
nullStr: null
```

**Why This Output:**

- `String.valueOf(42)` returns `"42"`.
- `wrapper.toString()` returns `"100"`.
- `"" + num` concatenates an empty string with `num`, producing `"42"`.
- `String.valueOf(nullWrapper)` returns `"null"` because `nullWrapper` is `null`.

**Real-World Cases:**

1. **Logging:** Converting objects to strings for log messages.
2. **UI display:** Formatting numeric values for user interfaces.
3. **String building:** Concatenating values into CSV, JSON, or XML.

**References:**

- Java API: `String.valueOf()` - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#valueOf(java.lang.Object)
- Java API: `Object.toString()` - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html#toString()

## Summary Table: Conversion Categories

| Conversion Type     | Direction             | Implicit?   | Risk                    | Example               |
| ------------------- | --------------------- | ----------- | ----------------------- | --------------------- |
| Widening Primitive  | Small → Large         | Yes         | Precision loss (rare)   | `int` → `double`      |
| Narrowing Primitive | Large → Small         | No (cast)   | Data loss               | `double` → `int`      |
| Numeric Promotion   | In expressions        | Yes         | Compile errors          | `byte + byte` → `int` |
| Upcasting           | Subclass → Superclass | Yes         | None                    | `Dog` → `Animal`      |
| Downcasting         | Superclass → Subclass | No (cast)   | `ClassCastException`    | `Animal` → `Dog`      |
| Autoboxing          | Primitive → Wrapper   | Yes         | Overhead, NPE (if null) | `int` → `Integer`     |
| Unboxing            | Wrapper → Primitive   | Yes         | `NullPointerException`  | `Integer` → `int`     |
| String Parsing      | String → Type         | No (method) | `NumberFormatException` | `"123"` → `int`       |
| String Conversion   | Type → String         | Yes         | None                    | `42` → `"42"`         |
