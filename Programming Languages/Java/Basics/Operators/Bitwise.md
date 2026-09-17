# Java Bitwise Operators: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Java bitwise operators are special symbols that perform operations on the individual bits of integer operands, treating numbers as sequences of binary digits (0s and 1s) rather than as whole decimal values.

**Technical Definition:** The Java Language Specification defines seven bitwise and shift operators for integral types: `&` (bitwise AND), `|` (bitwise inclusive OR), `^` (bitwise exclusive OR), `~` (unary bitwise complement), `<<` (signed left shift), `>>` (signed right shift), and `>>>` (unsigned right shift) . These operators operate on the binary representation of their operands, which are promoted to `int` if shorter than 32 bits. The `&`, `|`, and `^` operators can also operate on `boolean` operands, where they function as non-short-circuiting logical operators .

**Beginner-Friendly Explanation:** Bitwise operators let you work directly with the 1s and 0s that make up numbers inside the computer. Instead of thinking of "12" as twelve, you think of it as `1100`. These operators are useful when you need to pack multiple true/false values into a single number, work with hardware-level data, or perform certain mathematical tricks efficiently.

### Key Characteristics

- **Bit-Level Operations:** Each operator works on corresponding pairs of bits in the two operands (or on every bit for unary operators).
- **Integral Types Only:** Bitwise operations apply to `byte`, `short`, `int`, `long`, and `char` (but not `boolean` for shift operators) .
- **Binary Numeric Promotion:** Operands smaller than `int` are promoted to `int` before the operation .
- **Truth-Table Logic:** The `&`, `|`, `^`, and `~` operators follow the same logical rules as their boolean counterparts but applied bit-by-bit.
- **Shift Semantics:** Shifts move bits left or right, filling vacated positions based on the operator type.
- **Fast Execution:** Bitwise operations are among the fastest operations a CPU can perform, often completing in a single clock cycle .

### Prerequisites

- Understanding of Java primitive data types (`int`, `long`, `byte`)
- Familiarity with binary number representation and two's complement
- Knowledge of decimal-to-binary conversion
- Basic understanding of boolean logic (`true`/`false`)

### Related Programming Areas

- **Low-Level Programming:** Device drivers, embedded systems, and register manipulation
- **Performance Optimization:** Bit manipulation for speed-critical code
- **Data Compression:** Packing multiple values into fewer bits
- **Cryptography:** XOR-based encryption and hashing algorithms
- **Graphics and Game Development:** Color manipulation, collision detection, and bitmask operations

### Core Concepts / Features

---

## 1. Bitwise AND (`&`)

### Definitions

**Core Definition:** The bitwise AND operator `&` compares two integers bit-by-bit and produces a result where each bit is 1 only if both corresponding bits in the operands are 1.

**Technical Definition:** For integer operands, binary numeric promotion is applied, and the result is the bitwise AND of the two values . The truth table for each bit position is: `1 & 1 = 1`, and all other combinations yield `0`. When applied to `boolean` operands, `&` acts as a logical AND that always evaluates both operands (unlike `&&`) .

**Beginner-Friendly Explanation:** The `&` operator is like a strict filter: for each bit position, it only "passes through" a 1 if both numbers have a 1 there. It is commonly used to **mask** certain bits—keeping only the bits you are interested in and clearing the rest .

### Purposes

- **To extract specific bits** from a number using a mask (e.g., checking a particular flag).
- **To clear specific bits** by ANDing with a mask that has 0s in the target positions.
- **To test whether a flag is set** (e.g., `if ((flags & FLAG) != 0)`).
- **To perform bit masking** for packed data structures.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
result = operand1 & operand2;
```

**Syntax Breakdown:**
- `operand1`, `operand2`: Integer values (promoted to `int` if smaller).
- `&`: The bitwise AND operator.
- `result`: The integer result of the bitwise AND.

**Syntax Rules:**
- Both operands must be integer types or `boolean` types; mixing integer and boolean operands is a compile-time error .
- The operator is left-associative.
- For integer operands, binary numeric promotion is applied first.

**Constraints and Limitations:**
- **Not Short-Circuiting:** When used with `boolean` operands, `&` evaluates both sides; for short-circuit behavior, use `&&` .
- **No Floating-Point:** Bitwise operators do not work with `float` or `double`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Bitwise AND and Masking**

```java
// BitwiseAndDemo.java
public class BitwiseAndDemo {
    public static void main(String[] args) {
        // 13 in binary: 1101
        // 12 in binary: 1100
        int result = 13 & 12;
        System.out.println("13 & 12 = " + result); // 12 (binary 1100)
        
        // Using AND as a mask to extract low nibble (4 bits)
        int value = 0xABCD; // 1010101111001101
        int mask = 0x000F;  // 0000000000001111
        int lowNibble = value & mask;
        System.out.println("Low nibble: " + Integer.toHexString(lowNibble)); // d
        
        // Flag checking
        int READ = 1;    // 0001
        int WRITE = 2;   // 0010
        int EXECUTE = 4; // 0100
        int permissions = READ | WRITE; // 0011
        
        System.out.println("Has READ: " + ((permissions & READ) != 0));   // true
        System.out.println("Has EXECUTE: " + ((permissions & EXECUTE) != 0)); // false
    }
}
```

**Expected Output:**
```
13 & 12 = 12
Low nibble: d
Has READ: true
Has EXECUTE: false
```

**Why This Output Occurs:** `13 & 12` aligns the bits `1101` and `1100`. Only the highest two bits have 1s in both, yielding `1100` (12). The mask `0x000F` isolates the lowest 4 bits of `0xABCD`, which is `0xD`. Flag checks use `(permissions & FLAG) != 0` to test if a specific bit is set .

### Real-World Cases

- **Permission Systems:** Checking if a user has a specific permission bit set in a permission mask .
- **Graphics:** Extracting color channels (red, green, blue) from a packed pixel value.
- **Network Protocols:** Reading flag fields in packet headers.
- **Hardware Interfaces:** Reading status registers where each bit represents a device state.

**Explanation:** In a file system, read/write/execute permissions can be stored as three bits in a single integer, and `&` checks whether a particular permission is granted.

---

## 2. Bitwise OR (`|`)

### Definitions

**Core Definition:** The bitwise OR operator `|` compares two integers bit-by-bit and produces a result where each bit is 1 if at least one of the corresponding bits in the operands is 1.

**Technical Definition:** For integer operands, binary numeric promotion applies, and the result is the bitwise inclusive OR . The truth table: `0 | 0 = 0`, all other combinations yield `1`. With `boolean` operands, `|` acts as a logical OR that evaluates both sides (unlike `||`) .

**Beginner-Friendly Explanation:** The `|` operator is like a lenient gate: a bit position becomes 1 if either number has a 1 there. It is commonly used to **set** specific bits to 1 while leaving others unchanged .

### Purposes

- **To set specific bits** in a number to 1 (using a mask with 1s in the target positions).
- **To combine flags** into a single value (e.g., `READ | WRITE`).
- **To add permissions** or features without affecting existing ones.
- **To perform bit-packing** where multiple values are combined.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
result = operand1 | operand2;
```

**Syntax Rules:**
- Integer or `boolean` operands; binary numeric promotion applies .
- Left-associative.
- Non-short-circuiting when used with booleans.

**Constraints and Limitations:**
- Cannot mix integer and boolean operands.
- Does not short-circuit; use `||` for boolean short-circuit OR.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Setting Bits and Combining Flags**

```java
// BitwiseOrDemo.java
public class BitwiseOrDemo {
    public static void main(String[] args) {
        // 13 in binary: 1101
        // 12 in binary: 1100
        int result = 13 | 12;
        System.out.println("13 | 12 = " + result); // 13 (binary 1101)
        
        // Setting a specific bit using OR with a mask
        int number = 0; // 0000
        int bitPosition = 2;
        number = number | (1 << bitPosition); // Set bit 2
        System.out.println("After setting bit 2: " + number); // 4 (binary 100)
        
        // Combining multiple flags
        int FLAG_A = 1; // 0001
        int FLAG_B = 2; // 0010
        int FLAG_C = 4; // 0100
        int combined = FLAG_A | FLAG_B | FLAG_C; // 0111
        System.out.println("Combined flags: " + combined); // 7
    }
}
```

**Expected Output:**
```
13 | 12 = 13
After setting bit 2: 4
Combined flags: 7
```

**Why This Output Occurs:** `13 | 12` aligns `1101` and `1100`; any position with a 1 in either yields 1, giving `1101` (13). Setting bit 2 uses `1 << 2` (binary `100`) and ORs it with the original value . Combining flags ORs all the 1-bit masks together .

### Real-World Cases

- **Permission Assignment:** Granting multiple permissions by ORing their bit masks.
- **Option Flags:** Combining configuration options into a single integer.
- **Graphics:** Setting alpha channel bits in a pixel value.
- **Network Packets:** Setting flag bits in protocol headers.

**Explanation:** A file might be made both readable and writable by ORing the READ and WRITE permission flags, producing a single value that encodes both permissions.

---

## 3. Bitwise XOR (`^`)

### Definitions

**Core Definition:** The bitwise XOR (exclusive OR) operator `^` compares two integers bit-by-bit and produces a result where each bit is 1 if the corresponding bits are **different**, and 0 if they are the same.

**Technical Definition:** For integer operands, binary numeric promotion applies, and the result is the bitwise exclusive OR . Truth table: `0 ^ 0 = 0`, `0 ^ 1 = 1`, `1 ^ 0 = 1`, `1 ^ 1 = 0`. With `boolean` operands, `^` acts as logical XOR (true when operands differ) .

**Beginner-Friendly Explanation:** The `^` operator is like a "spot the difference" tool: a bit becomes 1 only if the two numbers disagree at that position. It is useful for toggling bits and for certain encryption tricks .

### Purposes

- **To toggle (flip) specific bits** by XORing with a mask of 1s.
- **To swap two values without a temporary variable** (a classic trick).
- **To implement simple encryption/decryption** (XOR with a key twice returns the original).
- **To detect differences** between two bit patterns.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
result = operand1 ^ operand2;
```

**Syntax Rules:**
- Integer or `boolean` operands .
- Left-associative.
- Non-short-circuiting with booleans.

**Constraints and Limitations:**
- XOR is its own inverse: `a ^ b ^ b = a`.
- Cannot mix integer and boolean operands.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Toggling Bits and XOR Swap**

```java
// BitwiseXorDemo.java
public class BitwiseXorDemo {
    public static void main(String[] args) {
        // 13 in binary: 1101
        // 12 in binary: 1100
        int result = 13 ^ 12;
        System.out.println("13 ^ 12 = " + result); // 1 (binary 0001)
        
        // Toggling a bit: XOR with 1 flips the bit
        int value = 5; // 0101
        int toggled = value ^ 1; // 0100 = 4
        System.out.println("5 ^ 1 = " + toggled); // 4
        System.out.println("4 ^ 1 = " + (toggled ^ 1)); // 5 (toggles back)
        
        // XOR swap (curiosity, not recommended for production)
        int x = 10;
        int y = 20;
        x = x ^ y;
        y = x ^ y;
        x = x ^ y;
        System.out.println("x=" + x + ", y=" + y); // x=20, y=10
    }
}
```

**Expected Output:**
```
13 ^ 12 = 1
5 ^ 1 = 4
4 ^ 1 = 5
x=20, y=10
```

**Why This Output Occurs:** `13 ^ 12` compares `1101` and `1100`; only the lowest bit differs, yielding `0001` (1). XORing with 1 toggles the lowest bit. The XOR swap works because `a ^ b ^ b = a`, so the intermediate values cancel out .

### Real-World Cases

- **Simple Encryption:** XORing plaintext with a key to produce ciphertext.
- **Parity Checking:** Detecting whether a bit pattern has an odd or even number of 1s.
- **Graphics:** Blending colors or creating visual effects.
- **Error Detection:** XOR checksums in data transmission.

**Explanation:** In a simple encryption scheme, each byte of a message is XORed with a key byte; applying the same XOR operation again recovers the original message.

---

## 4. Bitwise Complement (`~`)

### Definitions

**Core Definition:** The bitwise complement operator `~` is a unary operator that inverts every bit of its operand: 0 becomes 1, and 1 becomes 0.

**Technical Definition:** The `~` operator applies to a single integer operand. For any integer `N`, the result is `-(N + 1)` due to the two's complement representation of negative numbers . It is the only unary bitwise operator.

**Beginner-Friendly Explanation:** The `~` operator flips all the bits. In Java, because of how negative numbers are stored (two's complement), `~N` always equals `-N - 1`. For example, `~0` is `-1`, and `~5` is `-6`.

### Purposes

- **To invert all bits** in a bit pattern.
- **To compute negative values** in a bitwise context.
- **To create masks** by complementing a known pattern.
- **To implement bitwise NOT logic**.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
result = ~operand;
```

**Syntax Rules:**
- Operand must be an integer type (promoted to `int` if smaller).
- The result is always of type `int` or `long`.

**Constraints and Limitations:**
- Cannot be applied to `boolean` or reference types.
- The result is implementation-independent (two's complement is mandatory in Java).

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Complement and Negative Values**

```java
// BitwiseComplementDemo.java
public class BitwiseComplementDemo {
    public static void main(String[] args) {
        int value = 5; // 00000000 00000000 00000000 00000101
        int complement = ~value;
        System.out.println("~5 = " + complement); // -6
        
        // Verify: ~N = -(N + 1)
        System.out.println("-(5+1) = " + -(5 + 1)); // -6
        
        // Complement of 0
        System.out.println("~0 = " + ~0); // -1
        
        // Complement in a byte context (conceptual)
        byte b = 0; // 00000000
        // ~b promotes to int, so result is -1
        System.out.println("~(byte)0 = " + ~b); // -1
    }
}
```

**Expected Output:**
```
~5 = -6
-(5+1) = -6
~0 = -1
~(byte)0 = -1
```

**Why This Output Occurs:** In two's complement, `~5` flips `00000101` to `11111010`, which represents -6. The formula `~N = -(N + 1)` holds universally for Java integers .

### Real-World Cases

- **Creating Clear Masks:** `number & ~mask` clears the bits set in `mask` while preserving others.
- **Bitwise NOT Operations:** Inverting all flags in a configuration.
- **Low-Level Programming:** Complementing register values.

**Explanation:** To clear specific bits from a number, you can use `number & ~mask`, where `mask` has 1s in the positions you want to clear.

---

## 5. Left Shift (`<<`)

### Definitions

**Core Definition:** The left shift operator `<<` shifts the bits of its left operand to the left by the number of positions specified by the right operand, filling the vacated rightmost positions with 0s.

**Technical Definition:** The left shift operator performs a signed left shift. The bits shifted off the left end are discarded, and zeros are shifted in from the right . For each shift left by 1, the value is multiplied by 2 (barring overflow) .

**Beginner-Friendly Explanation:** Shifting left moves all bits toward the "big end." Each shift left doubles the number. For example, `5 << 1` is `10`, and `5 << 2` is `20`. It is a fast way to multiply by powers of 2.

### Purposes

- **To multiply by powers of 2** efficiently (e.g., `x << 3` is `x * 8`).
- **To create bit masks** at specific positions (e.g., `1 << 4` creates a mask with bit 4 set).
- **To pack data** into specific bit positions.
- **To perform fast arithmetic** in performance-critical code.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
result = operand1 << operand2;
```

**Syntax Rules:**
- `operand1`: The value to shift (integer type).
- `operand2`: The number of positions to shift (integer type).
- The result type is the promoted type of `operand1` .

**Constraints and Limitations:**
- **Overflow:** Shifting bits off the left end discards them permanently.
- **Sign Bit:** For `int`, shifting into bit 31 changes the sign.
- **Shift Amount:** The actual shift is `operand2 & 0x1F` for `int` (or `& 0x3F` for `long`) .

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Multiplication and Mask Creation**

```java
// LeftShiftDemo.java
public class LeftShiftDemo {
    public static void main(String[] args) {
        // Left shift as multiplication
        int value = 5;
        System.out.println("5 << 1 = " + (value << 1)); // 10
        System.out.println("5 << 2 = " + (value << 2)); // 20
        System.out.println("5 << 3 = " + (value << 3)); // 40
        
        // Creating a mask for bit position 4
        int bitPosition = 4;
        int mask = 1 << bitPosition; // 16 (binary 10000)
        System.out.println("Mask for bit 4: " + mask);
        
        // Combining shifts with OR to set multiple bits
        int combinedMask = (1 << 0) | (1 << 2) | (1 << 3); // bits 0,2,3
        System.out.println("Combined mask: " + combinedMask); // 13 (1101)
    }
}
```

**Expected Output:**
```
5 << 1 = 10
5 << 2 = 20
5 << 3 = 40
Mask for bit 4: 16
Combined mask: 13
```

**Why This Output Occurs:** Each left shift multiplies the value by 2. `1 << 4` places a 1 in bit position 4, creating the value 16. Combining masks with OR sets multiple bits simultaneously .

### Real-World Cases

- **Bit Mask Creation:** `1 << n` creates a mask for the nth bit.
- **Fast Multiplication:** In graphics and game loops, `x << 1` is faster than `x * 2` (though modern compilers optimize both).
- **Packing Data:** Shifting values into specific bit ranges for storage.

**Explanation:** In a permissions system, `1 << 3` creates a mask for the fourth permission bit (value 8), allowing up to 32 permissions in a single `int`.

---

## 6. Signed Right Shift (`>>`)

### Definitions

**Core Definition:** The signed right shift operator `>>` shifts the bits of its left operand to the right by the number of positions specified by the right operand, filling the vacated leftmost positions with the **sign bit** (0 for positive, 1 for negative).

**Technical Definition:** The `>>` operator performs an arithmetic right shift. If the left operand is positive, zeros are shifted in; if negative, ones are shifted in, preserving the sign . For each shift right by 1, the value is effectively divided by 2 (rounded toward negative infinity for negative numbers).

**Beginner-Friendly Explanation:** Shifting right moves all bits toward the "small end." Each shift right roughly halves the number. For positive numbers, `10 >> 1` is `5`. For negative numbers, the sign is preserved: `-8 >> 2` is `-2` .

### Purposes

- **To divide by powers of 2** efficiently (for positive numbers).
- **To extract specific bit ranges** from a packed value.
- **To preserve sign** when shifting negative numbers.
- **To unpack data** stored in specific bit positions.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
result = operand1 >> operand2;
```

**Syntax Rules:**
- The sign bit is replicated into vacated left positions .
- The shift amount is masked to `operand2 & 0x1F` for `int` (or `0x3F` for `long`).

**Constraints and Limitations:**
- **Not Division for Negatives:** `-5 >> 1` is `-3`, not `-2.5` truncated to `-2`.
- **Sign Extension:** Negative numbers stay negative after shifting.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Division and Sign Preservation**

```java
// SignedRightShiftDemo.java
public class SignedRightShiftDemo {
    public static void main(String[] args) {
        // Positive numbers: effectively divide by 2
        int positive = 20;
        System.out.println("20 >> 1 = " + (positive >> 1)); // 10
        System.out.println("20 >> 2 = " + (positive >> 2)); // 5
        
        // Negative numbers: sign is preserved
        int negative = -8;
        System.out.println("-8 >> 2 = " + (negative >> 2)); // -2
        System.out.println("-9 >> 2 = " + (-9 >> 2)); // -3 (rounds toward negative infinity)
        
        // Extracting bits from a packed value
        int packed = 0x1234; // 0001 0010 0011 0100
        int highByte = (packed >> 8) & 0xFF;
        System.out.println("High byte: " + Integer.toHexString(highByte)); // 12
    }
}
```

**Expected Output:**
```
20 >> 1 = 10
20 >> 2 = 5
-8 >> 2 = -2
-9 >> 2 = -3
High byte: 12
```

**Why This Output Occurs:** For positive numbers, `>>` effectively divides by 2. For negative numbers, the sign bit (1) is replicated, preserving negativity. `-9 >> 2` rounds toward negative infinity (-9/4 = -2.25, rounds to -3). Extracting the high byte uses shift and mask .

### Real-World Cases

- **Unpacking Data:** Extracting fields from packed integers (e.g., RGB values from a pixel).
- **Halving Values:** Fast division by 2 in performance-critical code.
- **Sign-Aware Arithmetic:** When negative values must stay negative after shifting.

**Explanation:** In an image processing application, `(pixel >> 16) & 0xFF` extracts the red channel from a 24-bit color value.

---

## 7. Unsigned Right Shift (`>>>`)

### Definitions

**Core Definition:** The unsigned right shift operator `>>>` shifts the bits of its left operand to the right by the specified number of positions, filling the vacated leftmost positions with **0s**, regardless of the sign.

**Technical Definition:** The `>>>` operator performs a logical right shift. It always fills the leftmost bits with zeros, treating the operand as an unsigned value . This is unique to Java (and C#); C and C++ do not have this operator for signed types.

**Beginner-Friendly Explanation:** Unlike `>>`, the `>>>` operator does not care about the sign. It always fills the left side with 0s. This is useful when you want to treat a negative number's bit pattern as a large positive number, or when you need to shift without preserving the sign.

### Purposes

- **To shift bits without sign extension** (treating the value as unsigned).
- **To extract bit patterns** from negative numbers as if they were positive.
- **To implement unsigned arithmetic** in Java (which lacks unsigned integer types).
- **To create masks** that require zero-fill behavior.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
result = operand1 >>> operand2;
```

**Syntax Rules:**
- Always fills vacated left positions with 0 .
- The shift amount is masked to `operand2 & 0x1F` for `int` (or `0x3F` for `long`).

**Constraints and Limitations:**
- **Different Result from `>>` for Negatives:** `-8 >> 2` is `-2`, but `-8 >>> 2` is a large positive number.
- **Not Arithmetic Division:** `>>>` does not perform division; it is purely a bit-level operation.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Unsigned Shift vs. Signed Shift**

```java
// UnsignedRightShiftDemo.java
public class UnsignedRightShiftDemo {
    public static void main(String[] args) {
        int negative = -8;
        
        // Signed right shift preserves sign
        System.out.println("-8 >> 2 = " + (negative >> 2)); // -2
        
        // Unsigned right shift fills with zeros
        System.out.println("-8 >>> 2 = " + (negative >>> 2)); // 1073741822
        
        // Positive numbers: same result for >> and >>>
        int positive = 20;
        System.out.println("20 >> 2 = " + (positive >> 2));  // 5
        System.out.println("20 >>> 2 = " + (positive >>> 2)); // 5
        
        // Using >>> to treat negative as unsigned for extraction
        int packed = -1; // all 1s
        int highByte = (packed >>> 24) & 0xFF;
        System.out.println("High byte of -1: " + Integer.toHexString(highByte)); // ff
    }
}
```

**Expected Output:**
```
-8 >> 2 = -2
-8 >>> 2 = 1073741822
20 >> 2 = 5
20 >>> 2 = 5
High byte of -1: ff
```

**Why This Output Occurs:** For `-8` (binary `11111111 11111111 11111111 11111000`), `>> 2` fills with 1s, giving `-2`. `>>> 2` fills with 0s, producing the large positive value `1073741822`. For positive numbers, both operators produce the same result because the sign bit is 0 .

### Real-World Cases

- **Unsigned Data Processing:** Treating Java's signed integers as unsigned when needed.
- **Cryptography:** Hash functions and ciphers often require unsigned shifts.
- **Network Protocols:** Reading unsigned fields from binary data.
- **Bit Manipulation:** Extracting bit patterns without sign interference.

**Explanation:** When parsing a network packet, a 32-bit unsigned integer might be stored as a Java `int`; using `>>>` allows correct extraction of all bits without negative-value interference.

---

## References Links

- Oracle. "Bitwise and Bit Shift Operators." The Java Tutorials. https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op3.html 
- Oracle. "Summary of Operators." The Java Tutorials. https://docs.oracle.com/javase/tutorial/java/nutsandbolts/opsummary.html 
- Oracle. "Chapter 15. Expressions." Java Language Specification, Java SE 26 Edition. https://docs.oracle.com/javase/specs/jls/se26/jls-15.html 
- Baeldung. "How to Set a Specific Bit of a Number in Java." https://www.baeldung.com/java-integer-specify-single-multiple-bits 
- Programiz. "Java Bitwise and Shift Operators." https://www.programiz.com/java-programming/bitwise-operators 
- Rutgers University. "Java Notes: Bitwise Operators." https://www.cs.rutgers.edu/courses/111/classes/fall_2011_venugopal/texts/notes-java/data/expressions/bitops.html 
- Indian Institute of Technology Bombay. "Shift and Bitwise Operators." https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/java/nutsandbolts/bitwise.html