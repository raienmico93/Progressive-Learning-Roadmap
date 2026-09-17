# Java String Basic Operations: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:**
Java String basic operations encompass the fundamental methods and operators used to manipulate, compare, and inspect string objects. These operations form the foundation of text processing in Java.

**Technical Definition:**
The `java.lang.String` class represents immutable character sequences. Basic operations include concatenation (creating new strings from existing ones), comparison (determining equality or ordering), and inspection (querying length and emptiness). All String methods return new String objects rather than modifying the original, due to String's immutable nature.

**Beginner-Friendly Explanation:**
Think of a String as a read-only text message. You cannot change the message itself, but you can create new messages by combining them, comparing them, or checking their properties (like how long they are or whether they're empty).

### Key Characteristics

- **Immutability:** String objects cannot be changed after creation. All "modifying" operations return new String objects.
- **Reference vs. Content:** The `==` operator compares references (memory addresses), while `.equals()` compares actual character content.
- **String Pool:** String literals are interned (cached) in a special memory area, which affects `==` behavior.
- **Unicode-based:** String operations work on UTF-16 code units, with comparison based on Unicode values.
- **Null Safety Varies:** Some methods handle null gracefully (`+` operator), while others throw `NullPointerException` (`concat()`, `join()`).

### Prerequisites

- Understanding of **primitive types** and **reference types** in Java.
- Familiarity with **object equality** concepts (`==` vs `.equals()`).
- Basic knowledge of **Unicode** and character encoding.
- Understanding of **immutability** and its implications.

### Related Programming Areas

- **Text Processing:** String operations are fundamental to parsing, formatting, and manipulating textual data.
- **Data Validation:** Checking `isEmpty()` and `isBlank()` is essential for input validation.
- **Sorting and Ordering:** `compareTo()` enables lexicographical sorting in collections.
- **String Building:** Understanding concatenation performance leads to proper use of `StringBuilder`.

## Core Concept 1: String Concatenation

**Core Definition:**
String concatenation is the process of combining two or more strings into a single new string.

**Technical Definition:**
Java provides multiple mechanisms for concatenation: the `+` and `+=` operators (syntactic sugar compiled to `StringBuilder` or `StringConcatFactory`), the `String.concat()` instance method, and the `String.join()` static method (Java 8+). All produce new String objects due to immutability.

**Beginner-Friendly Explanation:**
Concatenation is like gluing pieces of text together. Java gives you several tools for this: the `+` sign (easiest), the `.concat()` method (more explicit), and `.join()` (great for combining many strings with separators).

### 1: `+` Operator

**Definitions:**

- **Core Definition:** The `+` operator concatenates strings when at least one operand is a String.
- **Technical Definition:** The compiler transforms `a + b` into `new StringBuilder().append(a).append(b).toString()` in Java 8, or uses `invokedynamic` with `StringConcatFactory` in Java 9+ for more efficient concatenation .
- **Beginner-Friendly Explanation:** Just put `+` between two strings, and Java creates a new combined string.

**Purposes:**

- To combine strings with minimal syntax.
- To concatenate strings with non-string values (automatic conversion).
- To build messages from multiple parts.

**Syntax Structures and Rules:**

**General Syntax:**

```java
String result = string1 + string2;
string1 += string2;  // Compound assignment
```

**Syntax Rules:**

- At least one operand must be a String; otherwise, `+` performs numeric addition.
- Non-string operands are converted via `toString()`, or `"null"` if null .
- Left-associative: `a + b + c` evaluates as `(a + b) + c`.

**Constraints and Limitations:**

- Inefficient in loops (creates many intermediate objects).
- Java 9+ uses `StringConcatFactory` for potentially better performance.

**Multiple Annotated Complete Code Examples:**

**Example 1: Basic Concatenation**

```java
public class PlusOperatorDemo {
    public static void main(String[] args) {
        String first = "Hello";
        String second = "World";

        // Basic concatenation
        String combined = first + " " + second;
        System.out.println(combined);  // Hello World

        // Concatenation with primitive
        int number = 42;
        String message = "Value: " + number;
        System.out.println(message);  // Value: 42

        // Null handling
        String nullString = null;
        String withNull = "Result: " + nullString;
        System.out.println(withNull);  // Result: null
    }
}
```

**Expected Output:**

```
Hello World
Value: 42
Result: null
```

**Why This Output:**

- `first + " " + second` creates "Hello World".
- `number` (int 42) is converted to String "42".
- `nullString` is null; the `+` operator inserts "null" .

**Real-World Cases:**

1. **Log messages:** `"User " + username + " logged in at " + timestamp`
2. **Building URLs:** `baseUrl + "/api/" + endpoint`
3. **Report generation:** Combining labels with computed values.

**References:**

- Oracle Java Tutorials: Concatenating Strings - https://docs.oracle.com/javase/tutorial/java/data/strings.html

### 2: `String.concat()`

**Definitions:**

- **Core Definition:** A method that appends a string to the end of another, returning a new String.
- **Technical Definition:** `concat(String str)` creates a new character array of combined length, copies both strings, and returns a new String. If the argument is empty, returns `this` (the original string) .
- **Beginner-Friendly Explanation:** Like gluing a second piece of paper to the first, but you get a whole new piece—the originals stay unchanged.

**Purposes:**

- To explicitly concatenate without operator syntax.
- To work in environments where `+` is restricted (e.g., JSP EL).
- To concatenate efficiently when the appended string is empty.

**Syntax Structures and Rules:**

**General Syntax:**

```java
String result = original.concat(toAppend);
```

**Component Breakdown:**

- `original`: The string to which content is appended.
- `toAppend`: The string to append at the end.

**Syntax Rules:**

- Returns a new String if `toAppend` is non-empty.
- Throws `NullPointerException` if `toAppend` is null .
- More efficient than `+` when `toAppend` is empty (no new object created) .

**Constraints and Limitations:**

- NPE on null argument (unlike `+` operator).
- Inefficient for many concatenations (creates intermediate objects).

**Multiple Annotated Complete Code Examples:**

**Example 1: concat() vs + Operator**

```java
public class ConcatMethodDemo {
    public static void main(String[] args) {
        String base = "Hello";

        // concat() method
        String result1 = base.concat(" World");
        System.out.println(result1);  // Hello World

        // concat() returns original if appending empty
        String result2 = base.concat("");
        System.out.println(result2 == base);  // true (same object)

        // NPE on null
        try {
            String result3 = base.concat(null);
        } catch (NullPointerException e) {
            System.out.println("concat(null) throws NPE");
        }

        // + operator handles null
        String result4 = base + null;
        System.out.println(result4);  // Hellonull
    }
}
```

**Expected Output:**

```
Hello World
true
concat(null) throws NPE
Hellonull
```

**Why This Output:**

- `base.concat(" World")` creates a new String "Hello World".
- `base.concat("")` returns `this` (same object reference) .
- `concat(null)` throws NPE, while `+ null` produces "Hellonull" .

**Real-World Cases:**

1. **JSP EL:** `${firstName.concat(' ').concat(lastName)}` (EL doesn't support `+`) .
2. **Method chaining:** `path.concat("/").concat(fileName)`.
3. **Conditional concatenation:** When the suffix might be empty and you want to avoid creating a new object.

**References:**

- Java API: String.concat() - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#concat(java.lang.String)
- Java String concat source code analysis

### 3: `String.join()`

**Definitions:**

- **Core Definition:** A static utility method that joins multiple strings with a specified delimiter (Java 8+).
- **Technical Definition:** `String.join(CharSequence delimiter, CharSequence... elements)` or `String.join(CharSequence delimiter, Iterable<? extends CharSequence> elements)` creates a StringJoiner, adds each element with the delimiter, and returns the result .
- **Beginner-Friendly Explanation:** Like joining words with a separator, e.g., `"Hello" + ", " + "World"` becomes `String.join(", ", "Hello", "World")`.

**Purposes:**

- To join multiple strings with a consistent delimiter.
- To create CSV-like output from arrays or collections.
- To avoid manual delimiter management (no trailing delimiter issues).

**Syntax Structures and Rules:**

**General Syntax:**

```java
String result = String.join(delimiter, element1, element2, ...);
String result = String.join(delimiter, iterable);
```

**Component Breakdown:**

- `delimiter`: The separator placed between elements (must not be null).
- `elements`: The strings to join (varargs or Iterable).

**Syntax Rules:**

- Throws `NullPointerException` if delimiter is null.
- Throws `NullPointerException` if elements array is null.
- Null elements within the array are treated as "null" string .
- Returns empty string if no elements.

**Constraints and Limitations:**

- Cannot skip null elements (unlike Apache Commons `StringUtils.join()`).
- Delimiter must be non-null.

**Multiple Annotated Complete Code Examples:**

**Example 1: Basic join() Usage**

```java
import java.util.Arrays;
import java.util.List;

public class JoinMethodDemo {
    public static void main(String[] args) {
        // Join varargs
        String result1 = String.join("-", "2024", "01", "15");
        System.out.println(result1);  // 2024-01-15

        // Join with empty delimiter (no separator)
        String result2 = String.join("", "Hello", "World");
        System.out.println(result2);  // HelloWorld

        // Join a list
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        String result3 = String.join(", ", names);
        System.out.println(result3);  // Alice, Bob, Charlie

        // Null element becomes "null"
        String result4 = String.join(", ", "A", null, "C");
        System.out.println(result4);  // A, null, C
    }
}
```

**Expected Output:**

```
2024-01-15
HelloWorld
Alice, Bob, Charlie
A, null, C
```

**Why This Output:**

- First join uses "-" as delimiter between date parts.
- Second join uses empty delimiter, concatenating directly.
- Third join iterates the list, placing ", " between elements.
- Null element is converted to "null" string .

**Real-World Cases:**

1. **Building CSV:** `String.join(",", headers)`.
2. **File paths:** `String.join(File.separator, "home", "user", "docs")`.
3. **SQL IN clauses:** `"WHERE id IN (" + String.join(",", ids) + ")"`.

**References:**

- Java API: String.join() - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#join(java.lang.CharSequence,java.lang.CharSequence...)
- OpenJDK discussion on String.join null handling

## Core Concept 2: String Comparison

**Core Definition:**
String comparison determines whether two strings are equal or establishes their relative order. Java provides reference comparison (`==`), content comparison (`.equals()`), case-insensitive comparison (`.equalsIgnoreCase()`), and ordering (`.compareTo()`).

**Technical Definition:**
`==` compares object references (memory addresses). `String.equals()` is overridden to compare character sequences. `compareTo()` implements `Comparable<String>` and returns an integer indicating lexicographical order based on Unicode values.

**Beginner-Friendly Explanation:**
`==` asks "are these the same object?" while `.equals()` asks "do these contain the same text?" For sorting, `.compareTo()` tells you which string comes first alphabetically.

### 1: `==` Operator (Reference Equality)

**Definitions:**

- **Core Definition:** Checks whether two String variables refer to the exact same object in memory.
- **Technical Definition:** The `==` operator compares the reference values (pointers/addresses) of two String objects. It returns true only if both variables point to the same memory location .
- **Beginner-Friendly Explanation:** `==` asks "are these the same object?" not "do these look the same?" Two identical messages printed on different papers would be `false` with `==`.

**Purposes:**

- To check if two references point to the identical object.
- To verify object identity (rarely needed for Strings).

**Syntax Structures and Rules:**

**General Syntax:**

```java
boolean same = string1 == string2;
```

**Syntax Rules:**

- String literals are interned: `"Hello" == "Hello"` is `true` because literals are cached .
- `new String("Hello") == "Hello"` is `false` (different objects).
- Runtime-created strings (from input, files, etc.) are not interned.

**Constraints and Limitations:**

- **Never use `==` for content comparison** unless you specifically need reference equality.
- Results depend on string interning, which is unreliable for program logic.

**Multiple Annotated Complete Code Examples:**

**Example 1: `==` Behavior with Literals and new**

```java
public class ReferenceEqualityDemo {
    public static void main(String[] args) {
        // String literals are interned (cached)
        String s1 = "Hello";
        String s2 = "Hello";
        System.out.println("Literals: " + (s1 == s2));  // true

        // new String() creates a fresh object
        String s3 = new String("Hello");
        System.out.println("Literal vs new: " + (s1 == s3));  // false

        // Runtime-created string
        String s4 = "Hel" + "lo";  // Compile-time constant
        System.out.println("Concatenated literal: " + (s1 == s4));  // true

        // Runtime concatenation (not constant)
        String part = "lo";
        String s5 = "Hel" + part;
        System.out.println("Runtime concat: " + (s1 == s5));  // false
    }
}
```

**Expected Output:**

```
Literals: true
Literal vs new: false
Concatenated literal: true
Runtime concat: false
```

**Why This Output:**

- `"Hello"` literals are interned; `s1` and `s2` point to the same cached object.
- `new String("Hello")` creates a new heap object; different reference.
- `"Hel" + "lo"` is compile-time constant folding, resulting in the interned "Hello".
- `"Hel" + part` is runtime computation; creates a new String object .

**Real-World Cases:**

1. **Rarely used for Strings** — mainly for debugging or identity checks.
2. **Understanding interning** — helps explain why `==` sometimes works unexpectedly.

**References:**

- Stack Overflow: Using '==' instead of .equals for Java strings

### 2: `.equals()` (Content Equality)

**Definitions:**

- **Core Definition:** Compares two strings character by character to determine if they contain the same sequence.
- **Technical Definition:** `String.equals(Object)` is overridden from `Object` to perform content comparison. It returns true if the argument is a String with identical character sequence .
- **Beginner-Friendly Explanation:** `.equals()` asks "do these contain the same text?" regardless of whether they're the same object.

**Purposes:**

- To check if two strings have identical content.
- To validate user input against expected values.
- To perform string equality checks reliably.

**Syntax Structures and Rules:**

**General Syntax:**

```java
boolean equal = string1.equals(string2);
boolean equal = string1.equalsIgnoreCase(string2);  // Case-insensitive
```

**Syntax Rules:**

- Null-safe when called on non-null string: returns false if argument is null.
- Case-sensitive.
- Overrides `Object.equals()`; uses character-by-character comparison .

**Constraints and Limitations:**

- Not null-safe if the receiver is null: `null.equals(x)` throws NPE.
- For case-insensitive comparison, use `equalsIgnoreCase()`.

**Multiple Annotated Complete Code Examples:**

**Example 1: equals() vs ==**

```java
public class EqualsDemo {
    public static void main(String[] args) {
        String a = new String("Java");
        String b = new String("Java");

        // Reference comparison
        System.out.println("a == b: " + (a == b));  // false

        // Content comparison
        System.out.println("a.equals(b): " + a.equals(b));  // true

        // Null handling
        String c = null;
        System.out.println("a.equals(null): " + a.equals(c));  // false
        // System.out.println(c.equals(a));  // NPE if c is null
    }
}
```

**Expected Output:**

```
a == b: false
a.equals(b): true
a.equals(null): false
```

**Why This Output:**

- `a` and `b` are different objects (different memory addresses).
- `.equals()` compares character content: both are "Java".
- `.equals(null)` returns false (no NPE because receiver `a` is not null) .

**Real-World Cases:**

1. **Password validation:** `input.equals(storedHash)`.
2. **Command parsing:** `command.equals("start")`.
3. **Configuration checks:** `mode.equals("debug")`.

**References:**

- Java API: String.equals() - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#equals(java.lang.Object)
- Kiwix: Reference vs Content Comparison

### 3: `.equalsIgnoreCase()` (Case-Insensitive Equality)

**Definitions:**

- **Core Definition:** Compares two strings for content equality while ignoring uppercase/lowercase differences.
- **Technical Definition:** `equalsIgnoreCase(String)` performs character-by-character comparison using `Character.toUpperCase()` and `Character.toLowerCase()` for each character .
- **Beginner-Friendly Explanation:** Like `.equals()` but "Hello" and "HELLO" are considered equal.

**Purposes:**

- To validate user input where case doesn't matter.
- To compare commands or keywords case-insensitively.
- To check string equality in a locale-insensitive manner.

**Syntax Structures and Rules:**

**General Syntax:**

```java
boolean equal = string1.equalsIgnoreCase(string2);
```

**Syntax Rules:**

- Returns true if strings are equal ignoring case.
- Returns false if argument is null.
- Uses Unicode case folding rules.

**Constraints and Limitations:**

- Not locale-sensitive (for locale-aware, use `Collator`).
- Some Unicode characters may have unexpected case mappings.

**Multiple Annotated Complete Code Examples:**

**Example 1: Case-Insensitive Comparison**

```java
public class EqualsIgnoreCaseDemo {
    public static void main(String[] args) {
        String input = "YES";
        String expected = "yes";

        System.out.println("Exact: " + input.equals(expected));           // false
        System.out.println("IgnoreCase: " + input.equalsIgnoreCase(expected));  // true

        // User input validation
        String userCommand = "Quit";
        if (userCommand.equalsIgnoreCase("quit")) {
            System.out.println("Quitting...");
        }
    }
}
```

**Expected Output:**

```
Exact: false
IgnoreCase: true
Quitting...
```

**Why This Output:**

- `"YES".equals("yes")` is false (different characters).
- `"YES".equalsIgnoreCase("yes")` is true (case differences ignored).
- `"Quit".equalsIgnoreCase("quit")` is true.

**Real-World Cases:**

1. **User commands:** `if (input.equalsIgnoreCase("help"))`.
2. **File extension checks:** `fileName.equalsIgnoreCase("readme.txt")`.
3. **HTTP headers:** `header.equalsIgnoreCase("Content-Type")`.

**References:**

- Java API: String.equalsIgnoreCase() - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#equalsIgnoreCase(java.lang.String)

### 4: `.compareTo()` (Lexicographical Ordering)

**Definitions:**

- **Core Definition:** Compares two strings lexicographically (alphabetically) and returns an integer indicating their relative order.
- **Technical Definition:** Implements `Comparable<String>`. Returns a negative integer if this string precedes the argument, zero if equal, positive if it follows. Comparison is based on Unicode values of characters .
- **Beginner-Friendly Explanation:** Like dictionary order: `"apple".compareTo("banana")` is negative because "apple" comes first.

**Purposes:**

- To sort strings alphabetically.
- To implement custom ordering in collections.
- To determine which string is "less than" another.

**Syntax Structures and Rules:**

**General Syntax:**

```java
int result = string1.compareTo(string2);
```

**Return Value Meaning:**

- **Negative:** string1 < string2 (comes first)
- **Zero:** string1 equals string2
- **Positive:** string1 > string2 (comes after)

**Syntax Rules:**

- Based on Unicode values of characters, not locale rules .
- If one string is a prefix of the other, the shorter comes first .
- Throws NPE if argument is null.

**Constraints and Limitations:**

- Not locale-sensitive; use `Collator` for language-specific sorting.
- Case-sensitive: uppercase letters have lower Unicode values than lowercase .

**Multiple Annotated Complete Code Examples:**

**Example 1: compareTo() Return Values**

```java
public class CompareToDemo {
    public static void main(String[] args) {
        String a = "apple";
        String b = "banana";
        String c = "apple";
        String d = "app";

        System.out.println("a.compareTo(b): " + a.compareTo(b));  // negative
        System.out.println("a.compareTo(c): " + a.compareTo(c));  // 0
        System.out.println("b.compareTo(a): " + b.compareTo(a));  // positive
        System.out.println("a.compareTo(d): " + a.compareTo(d));  // positive (longer)

        // Case sensitivity: uppercase before lowercase
        System.out.println("'A'.compareTo('a'): " + "A".compareTo("a"));  // negative
    }
}
```

**Expected Output:**

```
a.compareTo(b): -1
a.compareTo(c): 0
b.compareTo(a): 1
a.compareTo(d): 2
'A'.compareTo('a'): -32
```

**Why This Output:**

- "apple" vs "banana": 'a' (97) vs 'b' (98), returns 97-98 = -1.
- "apple" vs "apple": identical, returns 0.
- "banana" vs "apple": 'b' (98) vs 'a' (97), returns 98-97 = 1.
- "apple" vs "app": shorter "app" is prefix; returns length difference 5-3 = 2 .
- "A" (65) vs "a" (97): returns 65-97 = -32 .

**Real-World Cases:**

1. **Sorting lists:** `Collections.sort(names)` uses `compareTo()`.
2. **Binary search:** Finding strings in sorted arrays.
3. **Ordering UI elements:** Sorting dropdown options alphabetically.

**References:**

- Java API: String.compareTo() - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#compareTo(java.lang.String)
- Stack Overflow: Lexicographic Order in Java

## Core Concept 3: String Length & Emptiness

**Core Definition:**
String length and emptiness operations determine the number of characters in a string and whether it contains meaningful content. Java provides `.length()`, `.isEmpty()`, and `.isBlank()` (Java 11+) for these purposes.

**Technical Definition:**
`.length()` returns the number of UTF-16 code units. `.isEmpty()` returns true only if length is 0. `.isBlank()` returns true if the string is empty or contains only whitespace codepoints as defined by `Character.isWhitespace(int)`.

**Beginner-Friendly Explanation:**
`length()` tells you how many characters. `isEmpty()` tells you if it's exactly zero characters. `isBlank()` tells you if it's empty _or_ just spaces/tabs/newlines.

### 1: `.length()`

**Definitions:**

- **Core Definition:** Returns the number of characters (UTF-16 code units) in the string.
- **Technical Definition:** Returns the `count` field of the String's internal character array. For supplementary characters (outside BMP), each surrogate pair counts as 2 .
- **Beginner-Friendly Explanation:** Counts the characters in the string.

**Purposes:**

- To determine string size for loops and validation.
- To check if a string has content.
- To compute substring boundaries.

**Syntax Structures and Rules:**

**General Syntax:**

```java
int length = string.length();
```

**Syntax Rules:**

- Returns an int ≥ 0.
- Not a field (unlike array `.length`); it's a method.
- Counts UTF-16 code units, not Unicode code points.

**Constraints and Limitations:**

- For emoji and supplementary characters, counts may be misleading (e.g., 😀 is 2).
- Use `codePointCount()` for actual Unicode character count.

**Multiple Annotated Complete Code Examples:**

**Example 1: length() Usage**

```java
public class LengthDemo {
    public static void main(String[] args) {
        String text = "Hello";
        System.out.println("Length: " + text.length());  // 5

        String empty = "";
        System.out.println("Empty length: " + empty.length());  // 0

        // Supplementary character (emoji)
        String emoji = "😀";
        System.out.println("Emoji length: " + emoji.length());  // 2 (surrogate pair)
        System.out.println("Code points: " + emoji.codePointCount(0, emoji.length()));  // 1
    }
}
```

**Expected Output:**

```
Length: 5
Empty length: 0
Emoji length: 2
Code points: 1
```

**Why This Output:**

- "Hello" has 5 UTF-16 code units.
- Empty string has 0.
- 😀 requires 2 UTF-16 code units (surrogate pair) but is 1 Unicode code point.

**Real-World Cases:**

1. **Input validation:** `if (input.length() > 100) reject();`
2. **Substring operations:** `text.substring(0, text.length() - 1)`.
3. **Password rules:** `if (password.length() < 8) reject();`.

**References:**

- Java API: String.length() - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#length()

### 2: `.isEmpty()`

**Definitions:**

- **Core Definition:** Returns true if and only if the string's length is exactly 0.
- **Technical Definition:** `isEmpty()` returns `value.length == 0` (or equivalent). It does not trim or check for whitespace .
- **Beginner-Friendly Explanation:** True only if the string has zero characters—not even a space.

**Purposes:**

- To check if a string has no characters.
- To avoid operations on empty strings.
- To validate required fields.

**Syntax Structures and Rules:**

**General Syntax:**

```java
boolean empty = string.isEmpty();
```

**Syntax Rules:**

- Returns `true` if `length() == 0`.
- Returns `false` for strings containing only whitespace.
- Null-safe? No—calling on null throws NPE.

**Constraints and Limitations:**

- Does not consider whitespace as empty.
- For whitespace-only check, use `isBlank()` (Java 11+).

**Multiple Annotated Complete Code Examples:**

**Example 1: isEmpty() vs Whitespace**

```java
public class IsEmptyDemo {
    public static void main(String[] args) {
        String empty = "";
        String space = " ";
        String text = "Hello";

        System.out.println("empty.isEmpty(): " + empty.isEmpty());  // true
        System.out.println("space.isEmpty(): " + space.isEmpty());  // false
        System.out.println("text.isEmpty(): " + text.isEmpty());    // false
    }
}
```

**Expected Output:**

```
empty.isEmpty(): true
space.isEmpty(): false
text.isEmpty(): false
```

**Why This Output:**

- `""` has length 0 → isEmpty true.
- `" "` has length 1 → isEmpty false.
- `"Hello"` has length 5 → isEmpty false .

**Real-World Cases:**

1. **Form validation:** `if (name.isEmpty()) showError();`.
2. **Early return:** `if (input.isEmpty()) return;`.
3. **Guard clauses:** Avoiding processing on empty strings.

**References:**

- Vultr Docs: Java String isEmpty()
- Java API: String.isEmpty() - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#isEmpty()

### 3: `.isBlank()` (Java 11+)

**Definitions:**

- **Core Definition:** Returns true if the string is empty or contains only whitespace characters.
- **Technical Definition:** `isBlank()` returns true if the string is empty or if all codepoints satisfy `Character.isWhitespace(int)`. This includes spaces, tabs, newlines, and other Unicode whitespace .
- **Beginner-Friendly Explanation:** Like `isEmpty()` but also considers strings with only spaces/tabs/newlines as "blank".

**Purposes:**

- To validate that input has actual content (not just spaces).
- To check for blank form fields.
- To avoid processing whitespace-only strings.

**Syntax Structures and Rules:**

**General Syntax:**

```java
boolean blank = string.isBlank();
```

**Syntax Rules:**

- Returns `true` for `""`, `" "`, `"\t"`, `"\n"`, `"  \t\n  "`.
- Returns `false` for `"Hello"`, `" Hello "` (contains non-whitespace).
- Introduced in Java 11.

**Constraints and Limitations:**

- Not available in Java 10 and earlier.
- For earlier versions, use `string.trim().isEmpty()` (but trim() removes only ASCII whitespace).

**Multiple Annotated Complete Code Examples:**

**Example 1: isBlank() vs isEmpty()**

```java
public class IsBlankDemo {
    public static void main(String[] args) {
        String[] tests = {"", " ", "  ", "\t", "\n", "Hello", " Hi "};

        for (String s : tests) {
            System.out.printf("'%s' - isEmpty: %b, isBlank: %b%n",
                s, s.isEmpty(), s.isBlank());
        }
    }
}
```

**Expected Output:**

```
'' - isEmpty: true, isBlank: true
' ' - isEmpty: false, isBlank: true
'  ' - isEmpty: false, isBlank: true
'\t' - isEmpty: false, isBlank: true
'\n' - isEmpty: false, isBlank: true
'Hello' - isEmpty: false, isBlank: false
' Hi ' - isEmpty: false, isBlank: false
```

**Why This Output:**

- `""`: empty and blank.
- `" "`, `"\t"`, `"\n"`: not empty (length > 0) but blank (only whitespace).
- `"Hello"`, `" Hi "`: contain non-whitespace, not blank .

**Real-World Cases:**

1. **Form validation:** `if (username.isBlank()) showError("Username required");`.
2. **Configuration parsing:** Skip blank lines in files.
3. **User input cleaning:** Reject inputs that are only spaces.

**References:**

- OpenJDK: String::isBlank JEP
- Stack Overflow: isBlank() since Java 11
- Microsoft Learn: String.IsBlank Property

## Summary Table: String Basic Operations

| Operation     | Method/Operator       | Key Point                                    | Java Version |
| ------------- | --------------------- | -------------------------------------------- | ------------ |
| Concatenation | `+`                   | Simple, null-safe, compiled to StringBuilder | 1.0+         |
| Concatenation | `.concat()`           | Explicit, NPE on null, efficient for empty   | 1.0+         |
| Concatenation | `String.join()`       | Delimiter-based, Java 8+                     | 8+           |
| Comparison    | `==`                  | Reference equality only                      | 1.0+         |
| Comparison    | `.equals()`           | Content equality                             | 1.0+         |
| Comparison    | `.equalsIgnoreCase()` | Case-insensitive content                     | 1.0+         |
| Comparison    | `.compareTo()`        | Lexicographical ordering                     | 1.0+         |
| Length        | `.length()`           | UTF-16 code units                            | 1.0+         |
| Emptiness     | `.isEmpty()`          | Length == 0                                  | 1.6+         |
| Blankness     | `.isBlank()`          | Empty or whitespace only                     | 11+          |

## References

1. Oracle Java Tutorials: Strings - https://docs.oracle.com/javase/tutorial/java/data/strings.html
2. Oracle Java API: String Class - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html
3. Educative: How to concatenate strings in Java - https://www.educative.io/answers/how-to-concatenate-strings-in-java
4. OpenJDK: String::isBlank JEP - https://bugs.openjdk.org/browse/JDK-8200437
5. Vultr Docs: Java String isEmpty() - https://docs.vultr.com/java/standard-library/java/lang/String/isEmpty
6. Stack Overflow: Lexicographic Order in Java - https://stackoverflow.com/questions/7874972/lexicographic-order-in-java
7. Stack Overflow: Using '==' instead of .equals for Java strings - https://stackoverflow.com/questions/3689952/using-instead-of-equals-for-java-strings
8. GitHub: Java 字符串拼接 - https://github.com/muskanmahajan37/blog-4/blob/master/后端/高效编程/03-Java-字符串拼接.md
