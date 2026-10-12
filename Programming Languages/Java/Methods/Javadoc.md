# Java Javadoc: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Javadoc is the standard documentation generation tool for Java, which extracts specially formatted comments from source code and produces HTML documentation describing the API.

**Technical Definition:** Javadoc is a tool that parses documentation comments (doc comments) written in a specific syntax within Java source files and generates HTML pages of API documentation. Documentation comments are recognized only when placed immediately before module, package, class, interface, constructor, method, or field declarations. The generated documentation describes the structure, types, methods, and behaviour of the API as declared in the source code.

**Beginner-Friendly Explanation:** Javadoc is like a built-in automatic documentation system for Java. You write special comments in your code using `/** ... */`, and Javadoc turns them into professional-looking web pages that describe what your classes and methods do. It's how Oracle generates the official Java API documentation that developers use every day.

### Key Characteristics

- **Source-Driven:** Documentation is extracted directly from source code comments, ensuring it stays synchronized with the code.
- **Standardized Syntax:** Uses a well-defined set of block tags (`@param`, `@return`, `@throws`) and inline tags (`{@link}`, `{@code}`).
- **HTML Output:** Produces HTML files that can be viewed in a web browser, navigated via hyperlinks, and searched.
- **Compiler-Integrated:** The `javadoc` tool calls part of the `javac` compiler to parse declarations, ensuring the output corresponds exactly with the actual implementation.
- **Extensible:** Supports custom doclets to generate output in formats other than HTML (XML, RTF, etc.).
- **Comment Inheritance:** Supports the `{@inheritDoc}` tag to inherit documentation from overridden or implemented methods.

### Prerequisites

- Basic knowledge of Java syntax, including classes, methods, fields, and constructors
- Familiarity with HTML basics (tags like `<p>`, `<ul>`, `<pre>`)
- Access to a Java Development Kit (JDK) installation
- Understanding of the command line (for direct `javadoc` tool usage) or a build tool like Maven or Gradle

### Related Programming Areas

- **API Design:** Javadoc is the primary mechanism for documenting public APIs in Java.
- **Build Tools:** Maven, Gradle, and Ant integrate Javadoc generation into the build lifecycle.
- **IDEs:** Eclipse, IntelliJ IDEA, and NetBeans provide Javadoc generation and preview features.
- **Documentation Generators:** Doclets can transform Javadoc comments into other formats (XML, Markdown, PDF).

### Core Concepts / Features

---

## 1. Javadoc

### Definitions

**Core Definition:** Javadoc is both the name of the tool (`javadoc`) and the name of the documentation format used to describe Java APIs.

**Technical Definition:** Javadoc is a documentation generator that processes source files ending in `.java` and produces HTML documentation. The `javadoc` command uses doclets to determine its output and uses the default Standard Doclet unless a custom doclet is specified with the `-doclet` option. It processes documentation comments placed immediately before declarations.

**Beginner-Friendly Explanation:** Javadoc is a tool that reads your Java code and the special comments you've written in it, then creates a website that documents your API. It's the same tool Oracle uses to create the official Java API documentation.

### Purposes

- To generate professional API documentation from source code
- To keep documentation synchronized with the code it describes
- To provide a standardized format for documenting Java APIs
- To enable developers to understand libraries without reading source code
- To support IDE tooltips and autocomplete features

### Syntax Rules and Structure

**Complete General Syntax (Command Line):**

```bash
javadoc [options] [packagenames] [sourcefiles] [@files]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[options]` | Command-line options that control output, source paths, and behaviour. |
| `[packagenames]` | Names of packages to document (separated by spaces). |
| `[sourcefiles]` | Individual source file names (may use wildcards). |
| `[@files]` | Files containing lists of package names or source files. |

**Syntax Rules:**

- The `javadoc` command requires access to the Java compiler and the classpath of the source files.
- Source files are processed only if the file name prefix is a valid class name, the path name is a valid package name, and the package statement matches.
- The tool produces one complete document per run; it does not perform incremental builds.

**Constraints and Limitations:**

- The `javadoc` tool must be able to find all referenced classes, including bootstrap classes and user classes.
- Documentation comments are not recognized inside method bodies.
- Only one documentation comment per declaration is recognized.

### Annotated Complete Step-by-Step Code Examples

**Example 1: A Simple Documented Class**

```java
// File: Calculator.java
/**
 * A simple calculator class that provides basic arithmetic operations.
 * <p>
 * This class demonstrates the use of Javadoc comments to document
 * a public API. Each method includes tags for parameters and return values.
 * </p>
 *
 * @author Jane Developer
 * @version 1.0
 * @since 1.0
 */
public class Calculator {

    /**
     * Adds two integers and returns the result.
     *
     * @param a the first integer to add
     * @param b the second integer to add
     * @return the sum of {@code a} and {@code b}
     */
    public int add(int a, int b) {
        return a + b;
    }

    /**
     * Divides the dividend by the divisor.
     *
     * @param dividend the number to be divided
     * @param divisor  the number to divide by
     * @return the quotient of {@code dividend} divided by {@code divisor}
     * @throws ArithmeticException if {@code divisor} is zero
     */
    public int divide(int dividend, int divisor) {
        if (divisor == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return dividend / divisor;
    }

    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println("5 + 3 = " + calc.add(5, 3));
        System.out.println("10 / 2 = " + calc.divide(10, 2));
    }
}
```

**Expected Output (when running the program):**
```
5 + 3 = 8
10 / 2 = 5
```

**Step-by-Step Explanation:**

1. The class-level Javadoc comment documents the class itself, including `@author`, `@version`, and `@since` tags.
2. The `add` method has a description, two `@param` tags (one for each parameter), and a `@return` tag.
3. The `divide` method documents a potential `ArithmeticException` using the `@throws` tag.
4. When `javadoc` is run on this file, it generates HTML pages showing all this documentation.

**Example 2: Generating Javadoc from the Command Line**

```bash
# Generate Javadoc for the Calculator class
javadoc -d docs Calculator.java
```

**Expected Output (partial):**
```
Loading source file Calculator.java...
Constructing Javadoc information...
Standard Doclet version 17.0.1
Building tree for all the packages and classes...
Generating docs/Calculator.html...
Generating docs/package-summary.html...
Generating docs/help-doc.html...
```

**Step-by-Step Explanation:**

1. The `-d docs` option specifies that the generated HTML files should be placed in the `docs` directory.
2. `Calculator.java` is the source file to document.
3. The tool processes the file and creates HTML documentation in the `docs` folder.
4. The generated `index.html` can be opened in a browser to view the documentation.

### Real-World Cases

- **Java Standard Library:** The official Java API documentation (e.g., `java.lang.String`) is generated with Javadoc.
- **Open-Source Libraries:** Projects like Spring, Hibernate, and Guava all publish Javadoc-generated documentation.
- **Internal APIs:** Companies use Javadoc to document internal libraries for their development teams.

---

## 2. Javadoc Comments

### Definitions

**Core Definition:** A Javadoc comment (also called a doc comment) is a special block comment that begins with `/**` and ends with `*/`, placed immediately before a declaration to be documented.

**Technical Definition:** Documentation comments are recognized only when placed immediately before module, package, class, interface, constructor, method, or field declarations. The overall form of a documentation comment is an initial (main) description, followed by a series of block tags that provide additional information about the declaration. Leading asterisks on each line are discarded, as are blanks and tabs that precede the initial asterisks.

**Beginner-Friendly Explanation:** A Javadoc comment is a special kind of comment that starts with `/**` instead of `/*`. Java ignores it for compilation, but the Javadoc tool reads it and turns it into documentation. It's like writing notes in your code that automatically become a manual.

### Purposes

- To document the purpose and behaviour of classes, methods, and fields
- To provide information about parameters, return values, and exceptions
- To enable automatic generation of API documentation
- To serve as inline documentation for developers using IDEs
- To establish a documentation contract for public APIs

### Syntax Rules and Structure

**Complete General Syntax:**

```java
/**
 * Main description of the declaration.
 * Can span multiple lines and include HTML.
 *
 * @param paramName description of the parameter
 * @return description of the return value
 * @throws ExceptionType description of when the exception is thrown
 */
public returnType methodName(parameters) {
    // implementation
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `/**` | Opening delimiter of the doc comment. |
| `*` | Leading asterisk on each line (optional but conventional). |
| Main description | The first sentence is the summary; subsequent text provides detail. |
| Block tags | Tags like `@param`, `@return`, `@throws` that begin at the start of a line. |
| `*/` | Closing delimiter of the doc comment. |

**Syntax Rules:**

- Doc comments must immediately precede the declaration they document.
- The first sentence of the main description is the summary sentence, used in summary tables.
- HTML content should be written in HTML 5.
- Headings in documentation comments for module, package, and type declarations should start at `<h2>`; for members, at `<h4>`.

**Constraints and Limitations:**

- Doc comments inside method bodies are ignored.
- Only one doc comment per declaration is recognized.
- The description cannot continue after the tag section begins.
- HTML content is not formally checked; errors may go undetected.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Documenting a Method with Full Tags**

```java
/**
 * Calculates the compound interest for a given principal, rate, and time.
 * <p>
 * The formula used is: {@code A = P * (1 + r)^t}
 * </p>
 *
 * @param principal the initial amount of money (must be positive)
 * @param rate      the annual interest rate as a decimal (e.g., 0.05 for 5%)
 * @param years     the number of years the money is invested
 * @return the final amount after compound interest
 * @throws IllegalArgumentException if {@code principal} is not positive
 * @see Math#pow(double, double)
 * @since 1.0
 */
public static double calculateCompoundInterest(double principal, double rate, int years) {
    if (principal <= 0) {
        throw new IllegalArgumentException("Principal must be positive");
    }
    return principal * Math.pow(1 + rate, years);
}
```

**Expected Output (Javadoc HTML rendering):**
The generated HTML page will show the description, parameter table, return value, exception, and a "See Also" link to `Math.pow`.

**Step-by-Step Explanation:**

1. The main description explains what the method does and includes the formula using `{@code}`.
2. `@param` tags document each parameter, including constraints.
3. `@return` describes the returned value.
4. `@throws` documents the exception thrown.
5. `@see` creates a link to a related method.
6. `@since` indicates the version in which the method was introduced.

**Example 2: Doc Comment with HTML Formatting**

```java
/**
 * Represents a shopping cart that holds items.
 * <p>
 * Example usage:
 * <pre>{@code
 * ShoppingCart cart = new ShoppingCart();
 * cart.addItem("Apple", 0.50);
 * }</pre>
 * </p>
 *
 * <h2>Thread Safety</h2>
 * <p>
 * This class is <strong>not</strong> thread-safe. External synchronization
 * is required when accessing it from multiple threads.
 * </p>
 */
public class ShoppingCart {
    // implementation
}
```

**Expected Output (Javadoc HTML rendering):**
The documentation will show a formatted example with monospaced code and a section heading "Thread Safety" with bold text.

**Step-by-Step Explanation:**

1. The `<p>` tags create separate paragraphs.
2. The `<pre>{@code ...}</pre>` block preserves formatting and displays code.
3. The `<h2>` tag creates a section heading (starting at `<h2>` for types).
4. The `<strong>` tag makes text bold.
5. The `{@code}` inline tag escapes special characters and renders in monospace.

### Real-World Cases

- **API Documentation:** Every public method in the Java standard library has a Javadoc comment.
- **Framework Documentation:** Spring's `@RestController` and `@RequestMapping` annotations are documented with Javadoc.
- **Internal Libraries:** Teams document shared utilities and services for other developers.

---

## 3. Javadoc Tags

### Definitions

**Core Definition:** Javadoc tags are special keywords within doc comments that begin with `@` (block tags) or `{@` (inline tags) and provide structured information about the documented declaration.

**Technical Definition:** A block tag has the form `@identifier content` and provides additional details that are incorporated into the generated documentation. Block tags must appear at the beginning of a line (ignoring leading asterisks and whitespace). Inline tags have the form `{@identifier content}` and can be placed anywhere in the main description or block tags.

**Beginner-Friendly Explanation:** Javadoc tags are like labels you put on different parts of your documentation. `@param` labels a parameter, `@return` labels the return value, and `{@link}` creates a clickable link. They tell Javadoc how to organize and display the information.

### Purposes

- To structure documentation in a consistent, machine-readable format
- To provide specific information about parameters, return values, and exceptions
- To create cross-references and links between related API elements
- To indicate version information, authorship, and deprecation status
- To embed code examples and literal text without HTML escaping issues

### Syntax Rules and Structure

**Complete General Syntax (Block Tags):**

```
@tagName content
```

**Complete General Syntax (Inline Tags):**

```
{@tagName content}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@tagName` | The tag identifier (case-sensitive). |
| `content` | The text associated with the tag; may span multiple lines. |

**Syntax Rules:**

- Block tags must appear at the start of a line (after leading asterisks and whitespace).
- Inline tags can appear anywhere within the text.
- Block tags should appear in a specific order: `@param`, `@return`, `@throws`, `@deprecated`, etc..
- Some tags can be repeated (e.g., `@param`), while others cannot (e.g., `@return`).

**Constraints and Limitations:**

- Using `@` at the beginning of a line when it is not intended as a tag requires using the HTML entity `&#064;`.
- Not all tags are valid in all contexts (e.g., `@return` is invalid for constructors and `void` methods).
- The `@exception` tag is a synonym for `@throws` and is deprecated in favour of `@throws`.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Block Tags for Methods**

```java
/**
 * Transfers funds from one account to another.
 *
 * @param fromAccount the account to withdraw from (must not be null)
 * @param toAccount   the account to deposit into (must not be null)
 * @param amount      the amount to transfer (must be positive)
 * @return the new balance of the source account
 * @throws IllegalArgumentException if amount is not positive
 * @throws NullPointerException     if either account is null
 * @see #withdraw(double)
 * @see #deposit(double)
 * @since 1.2
 * @deprecated Use {@link #transferFunds(Account, Account, double)} instead.
 */
@Deprecated
public double transfer(Account fromAccount, Account toAccount, double amount) {
    // implementation
}
```

**Expected Output (Javadoc HTML rendering):**
The generated documentation shows a parameter table, return value, exception descriptions, "See Also" links, and a deprecation notice.

**Step-by-Step Explanation:**

1. `@param` documents each parameter.
2. `@return` documents the return value.
3. `@throws` documents exceptions (used twice for two different exceptions).
4. `@see` creates links to related methods.
5. `@since` indicates the version introduced.
6. `@deprecated` marks the method as deprecated and suggests a replacement.

**Example 2: Inline Tags**

```java
/**
 * Returns the {@code Person} object with the given name.
 * <p>
 * This method uses a {@link java.util.HashMap} internally for fast lookup.
 * The name is compared using {@link String#equals(Object)}.
 * </p>
 * <p>
 * Example: {@code Person p = findPerson("Alice");}
 * </p>
 *
 * @param name the name to search for (must not be {@literal null})
 * @return the matching person, or {@code null} if not found
 */
public Person findPerson(String name) {
    // implementation
}
```

**Expected Output (Javadoc HTML rendering):**
- `{@code Person}` renders as `Person` in monospace.
- `{@link java.util.HashMap}` renders as a clickable link to the `HashMap` documentation.
- `{@link String#equals(Object)}` renders as a link to the `equals` method.
- `{@literal null}` renders as `null` without HTML interpretation.
- `{@code null}` renders as `null` in monospace.

**Step-by-Step Explanation:**

1. `{@code}` is used for code snippets and identifiers.
2. `{@link}` creates hyperlinks to other API elements.
3. `{@literal}` renders text without interpreting HTML or Javadoc tags.
4. Both `{@code}` and `{@literal}` prevent special characters from being processed.

### Tag Reference

**Common Block Tags:**

| Tag | Description | Applicable To |
|-----|-------------|---------------|
| `@param` | Documents a parameter | Methods, constructors, generic types |
| `@return` | Documents the return value | Methods (not void) |
| `@throws` | Documents an exception | Methods, constructors |
| `@exception` | Synonym for `@throws` | (Deprecated) |
| `@see` | Creates a "See Also" link | All declarations |
| `@since` | Indicates when the feature was added | All declarations |
| `@deprecated` | Marks as deprecated | All declarations |
| `@author` | Documents the author | Classes, interfaces |
| `@version` | Documents the version | Classes, interfaces |

**Common Inline Tags:**

| Tag | Description |
|-----|-------------|
| `{@link}` | Inserts an inline hyperlink |
| `{@code}` | Renders text in code font |
| `{@literal}` | Renders text without interpretation |
| `{@value}` | Displays the value of a constant field |
| `{@inheritDoc}` | Inherits documentation from a superclass or interface |

### Real-World Cases

- **`@param`:** Used in every public method of the JDK to describe parameters.
- **`@throws`:** Used in methods like `Integer.parseInt` to document `NumberFormatException`.
- **`@deprecated`:** Used extensively in the JDK to mark obsolete methods (e.g., `Date.getYear()`).
- **`{@link}`:** Used in JDK documentation to cross-reference related classes and methods.
- **`{@inheritDoc}`:** Used in collection classes to inherit documentation from interfaces.

---

## 4. Javadoc Generation

### Definitions

**Core Definition:** Javadoc generation is the process of running the `javadoc` tool (or a build-tool equivalent) to produce HTML documentation from source files.

**Technical Definition:** The `javadoc` tool processes source files and generates HTML output using the Standard Doclet by default. The tool can be invoked directly from the command line, or through build tools like Maven (`mvn javadoc:javadoc`) or Gradle (`gradle javadoc`). The generated output is placed in a destination directory specified by the `-d` option (command line), `target/site/apidocs` (Maven), or `build/docs/javadoc` (Gradle).

**Beginner-Friendly Explanation:** Javadoc generation is the act of running the Javadoc tool to turn your comments into a documentation website. You can do it by typing a command in the terminal, or by clicking a button in your IDE or build tool.

### Purposes

- To produce browsable HTML documentation from source code
- To integrate documentation generation into the build process
- To publish API documentation for users of a library
- To verify that documentation is complete and consistent

### Syntax Rules and Structure

**Command-Line Syntax:**

```bash
javadoc [options] [packagenames] [sourcefiles] [@files]
```

**Key Options:**

| Option | Description |
|--------|-------------|
| `-d directory` | Destination directory for HTML files. |
| `-author` | Includes `@author` text in the output. |
| `-version` | Includes `@version` text in the output. |
| `-link url` | Creates links to external Javadoc documentation. |
| `-subpackages` | Processes all subpackages recursively. |
| `-quiet` | Suppresses progress messages. |
| `-private` | Documents all classes and members (including private). |

**Maven Syntax:**

```bash
mvn javadoc:javadoc
```

**Gradle Syntax:**

```bash
gradle javadoc
```

**Syntax Rules:**

- The `javadoc` tool must be able to locate all referenced classes via the classpath.
- Source files must be valid Java files with proper package declarations.
- The output directory is created automatically if it does not exist.

**Constraints and Limitations:**

- The `javadoc` tool is not incremental; it generates a complete document each run.
- It may generate documentation for source files with incomplete or erroneous code.
- The tool performs only primitive checking of documentation comments.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Command-Line Javadoc Generation**

```bash
# Generate Javadoc for all classes in the com.example package
javadoc -d docs -author -version -link https://docs.oracle.com/en/java/javase/17/docs/api/ com.example
```

**Expected Output (partial):**
```
Generating docs/com/example/Calculator.html...
Generating docs/com/example/package-summary.html...
Generating docs/index.html...
Generating docs/help-doc.html...
```

**Step-by-Step Explanation:**

1. `-d docs` specifies the output directory.
2. `-author` includes author information in the output.
3. `-version` includes version information.
4. `-link` creates links to the official Java API documentation.
5. `com.example` is the package to document.

**Example 2: Maven Javadoc Configuration**

```xml
<!-- In pom.xml -->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>3.6.0</version>
            <configuration>
                <author>true</author>
                <version>true</version>
                <links>
                    <link>https://docs.oracle.com/en/java/javase/17/docs/api/</link>
                </links>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Expected Output:**
Running `mvn javadoc:javadoc` generates HTML documentation in `target/site/apidocs/`.

**Step-by-Step Explanation:**

1. The Maven Javadoc Plugin is configured in the `pom.xml`.
2. The `<author>` and `<version>` elements enable author and version information.
3. The `<links>` element creates external links to the Java API documentation.
4. Running `mvn javadoc:javadoc` executes the plugin and generates documentation.

### Real-World Cases

- **Open-Source Projects:** Projects publish Javadoc to GitHub Pages or their own websites.
- **CI/CD Pipelines:** Javadoc generation is often part of the build and deployment process.
- **IDE Integration:** IDEs provide "Generate Javadoc" actions and preview panes.

---

## 5. Javadoc Best Practices

### Definitions

**Core Definition:** Javadoc best practices are guidelines for writing effective, maintainable, and useful documentation comments.

**Technical Definition:** Best practices derive from authoritative sources including the Google Java Style Guide, Oracle's own documentation guidelines, and the Android API style guide. Key practices include writing complete sentences, documenting all parameters and return values, using `{@code}` for code, and maintaining consistency in tag order and formatting.

**Beginner-Friendly Explanation:** Javadoc best practices are like rules for writing good documentation. They help you write comments that are clear, complete, and actually useful to other developers.

### Purposes

- To ensure documentation is complete and accurate
- To improve readability and consistency across a codebase
- To make documentation useful for both humans and tools
- To reduce the time developers spend understanding APIs
- To maintain high-quality documentation as code evolves

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Write complete sentences | The first sentence should be a summary; use proper punctuation. |
| Document all parameters | Use `@param` for every parameter, including type parameters. |
| Document return values | Use `@return` for all non-void methods. |
| Document exceptions | Use `@throws` for all checked exceptions. |
| Use `{@code}` for code | Distinguish Java values and code from prose. |
| Use `{@link}` for references | Link to related classes and methods instead of plain text. |
| Maintain tag order | Use `@param`, `@return`, `@throws`, `@deprecated` in that order. |
| Avoid redundancy | Do not repeat what is obvious from the method signature. |
| Update documentation | Keep documentation synchronized with code changes. |

**Syntax Rules:**

- The first sentence of the main description is the summary fragment, which appears in indexes.
- Block tags should not have empty descriptions.
- Continuation lines for tags should be indented four or more spaces from the `@`.
- HTML content should be valid HTML 5.

**Constraints and Limitations:**

- Over-documenting trivial methods can clutter the documentation.
- Documentation comments are not checked for accuracy by the compiler.
- Poorly written documentation can be worse than none at all.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Well-Documented Method (Following Best Practices)**

```java
/**
 * Computes the greatest common divisor (GCD) of two positive integers
 * using the Euclidean algorithm.
 *
 * <p>The GCD is the largest positive integer that divides both numbers
 * without leaving a remainder. This method uses the iterative form of
 * the Euclidean algorithm.</p>
 *
 * @param a the first positive integer (must be &gt; 0)
 * @param b the second positive integer (must be &gt; 0)
 * @return the greatest common divisor of {@code a} and {@code b}
 * @throws IllegalArgumentException if either {@code a} or {@code b} is not positive
 * @see <a href="https://en.wikipedia.org/wiki/Euclidean_algorithm">Euclidean algorithm</a>
 * @since 1.0
 */
public static int gcd(int a, int b) {
    if (a <= 0 || b <= 0) {
        throw new IllegalArgumentException("Arguments must be positive");
    }
    while (b != 0) {
        int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}
```

**Expected Output (Javadoc HTML rendering):**
The generated page shows a clear description, a parameter table with constraints, a return value description, an exception description, a "See Also" link to Wikipedia, and a "Since" note.

**Step-by-Step Explanation:**

1. The main description explains what the method does and how it works.
2. The `<p>` tags separate paragraphs for readability.
3. `@param` tags include constraints in parentheses.
4. `@return` clearly describes the returned value.
5. `@throws` documents the exception condition.
6. `@see` creates a link to an external reference.
7. `@since` indicates the version introduced.

**Example 2: Poor vs. Good Documentation**

```java
// POOR: Redundant, incomplete, and unhelpful
/**
 * Gets the name.
 * @return the name
 */
public String getName() {
    return name;
}

// GOOD: Explains the purpose and any relevant context
/**
 * Returns the customer's full name as stored in the database.
 * <p>
 * The name is formatted as "Last, First" and may be empty if the
 * customer has not provided a name during registration.
 * </p>
 *
 * @return the formatted customer name, never {@code null} but possibly empty
 * @see #setName(String)
 */
public String getName() {
    return name;
}
```

**Expected Output (Javadoc HTML rendering):**
The poor version produces a trivial, unhelpful entry. The good version produces a useful description with context and a related method link.

**Step-by-Step Explanation:**

1. The poor version merely restates the method name and return type.
2. The good version explains what the name represents, its format, and possible empty values.
3. The `@see` tag links to the related setter method.

### Real-World Cases

- **Google Java Style Guide:** Specifies tag order and formatting rules for Javadoc.
- **Android API Guidelines:** Require `@param` and `@return` documentation for all public methods.
- **JDK Source Code:** Demonstrates best practices in the Java standard library.
- **Open-Source Projects:** Well-maintained projects like Guava and Spring enforce Javadoc standards.

---

## References Links

- javadoc - Oracle Java Documentation - https://docs.oracle.com/en/java/javase/17/docs/specs/man/javadoc.html
- Documentation Comment Specification for the Standard Doclet (JDK 17) - https://docs.oracle.com/en/java/javase/17/docs/specs/javadoc/doc-comment-spec.html
- Javadoc Tool Home Page - https://docs.oracle.com/javase/8/docs/technotes/guides/javadoc/
- How to Write Doc Comments for the Javadoc Tool - https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html
- Google Java Style Guide — Javadoc - https://google.github.io/styleguide/javaguide.html#s7-javadoc
- Android API Reference Style Guide - https://source.android.com/docs/setup/contribute/api-guidelines
- Gradle Javadoc Style Guide - https://github.com/gradle/gradle/blob/master/CONTRIBUTING.md
- Maven Javadoc Plugin - https://maven.apache.org/plugins/maven-javadoc-plugin/
- Javadoc: The Java API Documentation Generator Explained and Mastered - https://www.cleverence.com/articles/tech-blogs/javadoc-the-java-api-documentation-generator-explained-and-mastered
- Defining Methods — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html