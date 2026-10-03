# Java Annotations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

A Java annotation is a form of metadata that provides data about a program but has no direct effect on the operation of the code it annotates. Annotations are used to associate information with program elements (classes, methods, fields, parameters) and can be processed at compile time or runtime.

### Technical Definition

An annotation is a special kind of interface introduced in Java 5 (JSR 175) that declares a new annotation type. An annotation interface is declared with the `@interface` keyword, and its elements are method declarations with no parameters and restricted return types. Annotations are retained according to their `@Retention` policy (SOURCE, CLASS, or RUNTIME) and restricted to specific syntactic locations via `@Target`. At runtime, annotations retained with `RUNTIME` policy can be read reflectively. At compile time, annotations can be processed by the Pluggable Annotation Processing API (JSR 269), which allows processors to generate source files, validate schemas, or emit diagnostics.

### Beginner-Friendly Explanation

Think of annotations as sticky notes you attach to parts of your code. A sticky note on a method might say "this overrides a parent method" or "this is deprecated, don't use it." Some sticky notes are just for the compiler (they disappear after compilation), while others stay on the compiled code and can be read at runtime by frameworks like Spring or Hibernate. You can even create your own custom sticky notes with specific instructions for your own tools.

### Key Characteristics

- **Metadata**: Annotations do not change program semantics; they provide information.
- **Declarative**: They are placed before declarations, similar to modifiers.
- **Retention-Aware**: Their lifecycle is controlled by the `@Retention` policy.
- **Target-Restricted**: `@Target` limits where an annotation can be applied.
- **Reflectable**: RUNTIME-retained annotations are accessible via reflection.
- **Compile-Time Processable**: Annotation processors can generate code or validate at compile time.

### Prerequisites

- Basic Java programming knowledge (interfaces, classes, methods).
- Familiarity with the Java compiler (`javac`).
- Understanding of reflection (for runtime annotation processing).
- Awareness of the `java.lang.annotation` package.

### Related Programming Areas

- **Reflection API**: Reading annotations at runtime.
- **Annotation Processing**: Compile-time code generation and validation.
- **Frameworks**: Spring, Hibernate, JUnit rely heavily on annotations.
- **Java Module System**: Annotation processing in modular applications.
- **Project Checker Framework**: Type annotations for static analysis.

### Core Concepts Overview

1. **Annotation Declaration**: Creating metadata interfaces with `@interface`.
2. **Built-in Annotations**: `@Override`, `@Deprecated`, `@SuppressWarnings`, `@FunctionalInterface`, `@SafeVarargs`, `@Serial`.
3. **Custom Annotations**: Domain-specific metadata for frameworks and tools.
4. **Retention Policies**: Controlling annotation lifecycle with `@Retention`.
5. **Targets**: Restricting placement with `@Target` and `ElementType`.
6. **Annotation Processing**: Compile-time hooks via `javax.annotation.processing`.

---

## Core Concept 1: Annotation Declaration

### Definitions

**Core Definition**: Annotation declaration is the process of creating a new annotation type using the `@interface` keyword, defining its elements, default values, and allowed data types.

**Technical Definition**: An annotation interface declaration specifies a new annotation interface, a specialized kind of interface. To distinguish it from a normal interface declaration, the keyword `interface` is preceded by an at sign (`@`). Annotation interfaces are never generic and cannot declare type parameters. Methods declared in an annotation interface must not have formal parameters, type parameters, or a `throws` clause; their return types are limited to primitives, `String`, `Class` (or invocation of `Class`), enum types, annotation types, or arrays of these types. Methods may have default values, but `null` is expressly forbidden as a default.

**Beginner-Friendly Explanation**: Declaring an annotation is like designing a custom form. The `@interface` keyword is the form template. Each element is a blank to fill in (like "name" or "value"). You can specify default answers so some blanks are optional. The type of each blank is restricted—you can't ask for arbitrary objects, only simple types like text, numbers, or other annotations.

### Purposes

- To define structured metadata that can be attached to Java elements.
- To create domain-specific markers for frameworks and tools.
- To provide compile-time or runtime configuration information.
- To enable declarative programming styles (e.g., `@Test`, `@Entity`).
- To support code generation and validation via annotation processors.
- To establish a contract between annotation authors and processors.

### Syntax Rules and Structure

#### Complete General Syntax: Annotation Declaration

```
ANNOTATION DECLARATION SYNTAX
│
├── Declaration
│   ├── modifiers @interface AnnotationName {
│   │       elementType elementName();
│   │       elementType elementName() default value;
│   │   }
│   └── The @interface is a distinct token from interface
│
├── Element Types (allowed return types)
│   ├── primitives (int, long, double, boolean, etc.)
│   ├── String
│   ├── Class (or Class<? extends X>)
│   ├── enum types
│   ├── annotation types
│   └── arrays of the above
│
├── Default Values
│   ├── type elementName() default value;
│   ├── null is NOT allowed
│   └── Default makes the element optional
│
└── Meta-Annotations (applied to the annotation declaration)
    ├── @Retention(RetentionPolicy.RUNTIME)
    ├── @Target(ElementType.METHOD)
    └── @Documented
```

#### Component Breakdown

| Component | Purpose | Restrictions |
|-----------|---------|--------------|
| `@interface` | Declares annotation type | Distinct from `interface` |
| Element method | Defines a data element | No params, no throws |
| Element type | Return type of element | Limited set of types |
| `default` | Provides fallback value | `null` forbidden |
| Meta-annotations | Configure annotation behavior | See Retention and Target |

#### Syntax Rules

- Use `@interface` (at-sign followed by `interface`) to declare an annotation type.
- Annotation declarations can be top-level or member interfaces, but not local interfaces.
- Elements are declared as methods with no parameters.
- Element return types are restricted to primitives, `String`, `Class`, enums, annotations, or arrays of these.
- Default values make elements optional; without a default, the element must be supplied.
- `null` cannot be used as a default or explicit value.
- By convention, a single-element annotation's element is named `value`.

#### Constraints and Limitations

- Annotation interfaces cannot be generic.
- Annotation interfaces cannot extend other interfaces or classes.
- Element methods cannot have parameters, type parameters, or `throws` clauses.
- `null` is not a legal element value.
- Annotation interfaces cannot be declared as local interfaces (inside method bodies).
- Modifiers `sealed` and `non-sealed` are prohibited on annotation declarations.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Annotation Declaration

**Setup Guide**: Save as `BasicAnnotationDemo.java`, compile with `javac`, and run with `java`.

```java
// BasicAnnotationDemo.java
import java.lang.annotation.*;

// Declare a custom annotation with two elements
@interface Author {
    String name();                    // Required element
    String date() default "unknown";  // Optional element with default
}

// Apply the annotation
@Author(name = "Alice", date = "2026-09-28")
class Document {
    @Author(name = "Bob")
    public void method() {}
}

public class BasicAnnotationDemo {
    public static void main(String[] args) throws Exception {
        System.out.println("=== Basic Annotation Declaration ===\n");
        
        // Read annotation from class
        Class<?> clazz = Document.class;
        Author classAuthor = clazz.getAnnotation(Author.class);
        System.out.println("Class author : " + classAuthor.name());
        System.out.println("Class date   : " + classAuthor.date());
        // Class author : Alice
        // Class date   : 2026-09-28
        
        // Read annotation from method
        Author methodAuthor = clazz.getMethod("method")
            .getAnnotation(Author.class);
        System.out.println("\nMethod author       : " + methodAuthor.name());
        System.out.println("Method date (default) : " + methodAuthor.date());
        // Method author: Bob
        // Method date (default): unknown
    }
}
```

**Expected Output**:
```
=== Basic Annotation Declaration ===

Class author : Alice
Class date   : 2026-09-28

Method author         : Bob
Method date (default) : unknown
```

**Why This Output**: The `@Author` annotation is declared with `@interface`. The `name` element is required; `date` has a default value. The annotation is applied to both the class and a method. At runtime, `getAnnotation()` retrieves the annotation, and `name()` and `date()` access the element values. The method's `date` uses the default `"unknown"` because it was not specified.

---

### Real-World Cases

- **Spring Framework**: `@Component`, `@Service`, `@Repository` are custom annotations that mark classes for component scanning.
- **JUnit**: `@Test`, `@BeforeEach`, `@AfterEach` annotate test methods for discovery and execution.
- **Hibernate/JPA**: `@Entity`, `@Table`, `@Column` map classes and fields to database structures.
- **Jackson**: `@JsonProperty`, `@JsonIgnore` customize JSON serialization.

### References

- Chapter 9. Interfaces: Annotation Interfaces - https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html#jls-9.6
- Annotation Type Declarations - Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/annotations/declaring.html

---

## Core Concept 2: Built-in Annotations

### Definitions

**Core Definition**: Built-in annotations are predefined annotation types provided by the Java platform in the `java.lang` package (and `java.io` for `@Serial`) that serve common compiler-checking and documentation purposes.

**Technical Definition**: The Java platform defines several annotation types used by the compiler or for documentation. In `java.lang`: `@Override` (informs the compiler that a method overrides a superclass method), `@Deprecated` (indicates the marked element should no longer be used), `@SuppressWarnings` (suppresses specific compiler warnings), `@FunctionalInterface` (indicates an interface is intended to be a functional interface), and `@SafeVarargs` (asserts that a varargs method is safe to use). In `java.io`: `@Serial` (marks serialization-related fields and methods). Since Java 9, `@Deprecated` has `since` and `forRemoval` elements.

**Beginner-Friendly Explanation**: These are the "official sticky notes" that come with Java. `@Override` is a safety check: it tells the compiler "I think I'm overriding a parent method—please verify." `@Deprecated` is a warning label: "This is old, avoid using it." `@SuppressWarnings` is a "silence this specific warning" note. `@FunctionalInterface` marks interfaces meant for lambdas.

### Purposes

- To provide compiler-checked correctness guarantees for common patterns.
- To document deprecation and migration paths.
- To suppress specific compiler warnings when safe.
- To mark functional interfaces for lambda compatibility.
- To validate varargs methods for unchecked warnings.
- To enable compile-time checking of serialization declarations.

### Syntax Rules and Structure

#### Complete General Syntax: Built-in Annotations

```
BUILT-IN ANNOTATIONS SYNTAX
│
├── @Override
│   └── Target: METHOD
│       └── Compiler error if method doesn't override
│
├── @Deprecated
│   ├── Target: CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, MODULE, PARAMETER, TYPE
│   ├── Elements: since (String, default ""), forRemoval (boolean, default false)
│   └── Compiler warning on use
│
├── @SuppressWarnings
│   ├── Target: TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE
│   ├── Element: value (String[] or String)
│   └── Suppresses specified warning categories
│
├── @FunctionalInterface
│   ├── Target: TYPE
│   └── Compiler error if interface has != 1 abstract method
│
├── @SafeVarargs
│   ├── Target: METHOD, CONSTRUCTOR
│   └── Suppresses unchecked warnings for varargs
│
└── @Serial
    ├── Target: FIELD, METHOD
    └── Compiler warning if not a valid serialization member
```

#### Component Breakdown

| Annotation | Package | Primary Use | Compiler Action |
|-----------|---------|-------------|-----------------|
| `@Override` | `java.lang` | Override check | Error if no override |
| `@Deprecated` | `java.lang` | Mark obsolete | Warning on use |
| `@SuppressWarnings` | `java.lang` | Silence warnings | Suppresses specified |
| `@FunctionalInterface` | `java.lang` | Mark functional interface | Error if not functional |
| `@SafeVarargs` | `java.lang` | Assert varargs safety | Suppresses unchecked |
| `@Serial` | `java.io` | Mark serialization member | Warning if misdeclared |

#### Syntax Rules

- `@Override` should be used on methods that override superclass methods; the compiler verifies.
- `@Deprecated` can include `since = "version"` and `forRemoval = true` since Java 9.
- `@SuppressWarnings` takes a `String` or `String[]` of warning category names (e.g., `"unchecked"`, `"deprecation"`).
- `@FunctionalInterface` requires exactly one abstract method; default and static methods are allowed.
- `@SafeVarargs` can only be applied to `static`, `final`, or `private` methods and constructors.
- `@Serial` is applicable to the five serialization methods and two fields defined by the Serialization Specification.

#### Constraints and Limitations

- `@Override` on a method that doesn't override causes a compile-time error.
- `@Deprecated` alone does not prevent use; it only generates warnings.
- `@SuppressWarnings` should be used sparingly and with specific categories.
- `@FunctionalInterface` is informative; the compiler enforces the single abstract method rule regardless of the annotation.
- `@SafeVarargs` is a programmer assertion; incorrect use can lead to heap pollution.
- `@Serial` requires Java 14+.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Built-in Annotations in Action

**Setup Guide**: Save as `BuiltInAnnotationsDemo.java`, compile with `javac -Xlint:all`, and run with `java`.

```java
// BuiltInAnnotationsDemo.java
public class BuiltInAnnotationsDemo {

    // @FunctionalInterface: ensures exactly one abstract method
    @FunctionalInterface
    interface Calculator {
        int calculate(int a, int b);
    }

    static class MathOperations {
        // @Deprecated with since and forRemoval
        @Deprecated(since = "1.0", forRemoval = true)
        public int oldAdd(int a, int b) {
            return a + b;
        }

        public int newAdd(int a, int b) {
            return a + b;
        }
    }

    static class Child extends MathOperations {
        @Override
        public int newAdd(int a, int b) {
            return super.newAdd(a, b);
        }
    }

    // @SafeVarargs: asserts no unsafe operations on varargs
    @SafeVarargs
    public static <T> void printAll(T... items) {
        for (T item : items) {
            System.out.println("  " + item);
        }
    }

    // @SuppressWarnings: suppresses deprecation warning
    @SuppressWarnings("deprecation")
    public static void useDeprecated() {
        MathOperations ops = new MathOperations();
        System.out.println("oldAdd(2, 3) = " + ops.oldAdd(2, 3));
    }

    public static void main(String[] args) {
        System.out.println("=== Built-in Annotations Demo ===\n");

        // @FunctionalInterface usage with lambda
        Calculator adder = (a, b) -> a + b;
        System.out.println("Lambda result: " + adder.calculate(5, 7));

        // @Deprecated usage (suppressed)
        useDeprecated();

        // @SafeVarargs usage
        System.out.println("\nVarargs:");
        printAll("Java", "Annotations", "Demo");

        System.out.println("\nAll built-in annotations demonstrated.");
    }
}
```

**Expected Output**:
```
=== Built-in Annotations Demo ===

Lambda result: 12
oldAdd(2, 3) = 5

Varargs:
  Java
  Annotations
  Demo

All built-in annotations demonstrated.
```

**Why This Output**: `@FunctionalInterface` allows the lambda `(a, b) -> a + b` to implement `Calculator`. `@Deprecated` marks `oldAdd` but the warning is suppressed by `@SuppressWarnings("deprecation")` in `useDeprecated`. `@SafeVarargs` suppresses unchecked warnings for `printAll`. `@Override` on `Child.newAdd` is verified by the compiler.

---

### Real-World Cases

- **API Evolution**: `@Deprecated(since="9", forRemoval=true)` is used throughout the JDK to signal removal intent.
- **Lambda Support**: `@FunctionalInterface` is used on `Runnable`, `Comparator`, `Function`, and other JDK interfaces.
- **Serialization**: `@Serial` is used in JDK classes like `ArrayList` and `HashMap` to mark `serialVersionUID` and serialization methods.
- **Legacy Code**: `@SuppressWarnings("unchecked")` is used when interfacing with pre-generics code.

### References

- Predefined Annotation Types - Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/annotations/predefined.html
- Annotation Interface Serial - Java SE 21 - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Serial.html
- Deprecated - Java SE 9 - https://docs.oracle.com/javase/9/docs/api/java/lang/Deprecated.html

---

## Core Concept 3: Custom Annotations

### Definitions

**Core Definition**: Custom annotations are user-defined annotation types created with `@interface` to carry domain-specific metadata that configures framework runtimes, validation engines, or serialization matrices.

**Technical Definition**: A custom annotation is an annotation interface declared by the developer. It typically includes meta-annotations such as `@Retention(RetentionPolicy.RUNTIME)` (for reflection-based processing) and `@Target(...)` (to restrict placement). Custom annotations may have elements with default values. Frameworks process custom annotations via reflection (runtime) or annotation processors (compile-time). Common use cases include dependency injection (`@Autowired`), validation (`@NotNull`), and serialization configuration (`@JsonProperty`).

**Beginner-Friendly Explanation**: Custom annotations are your own sticky notes. If you're building a framework that needs to know which methods should be logged or which fields should be validated, you create annotations like `@Loggable` or `@Validate`. Your framework then reads these annotations at runtime (via reflection) or compile time (via processors) and acts accordingly.

### Purposes

- To define domain-specific metadata for framework configuration.
- To enable declarative programming patterns in application code.
- To support runtime processing via reflection.
- To enable compile-time code generation and validation.
- To decouple configuration from implementation logic.
- To provide self-documenting code through semantic markers.

### Syntax Rules and Structure

#### Complete General Syntax: Custom Annotation with Meta-Annotations

```
CUSTOM ANNOTATION SYNTAX
│
├── Declaration with Meta-Annotations
│   ├── @Retention(RetentionPolicy.RUNTIME)
│   ├── @Target(ElementType.METHOD)
│   ├── @Documented
│   └── public @interface MyAnnotation {
│           String value() default "";
│           int priority() default 0;
│       }
│
├── Applying the Annotation
│   ├── @MyAnnotation(value = "process", priority = 1)
│   └── public void myMethod() { ... }
│
└── Processing at Runtime (Reflection)
    ├── Method method = clazz.getMethod("myMethod");
    ├── MyAnnotation ann = method.getAnnotation(MyAnnotation.class);
    └── if (ann != null) { use ann.value(), ann.priority() }
```

#### Component Breakdown

| Meta-Annotation | Purpose | Common Value |
|----------------|---------|--------------|
| `@Retention` | Lifecycle | `RUNTIME` for reflection |
| `@Target` | Placement restriction | `METHOD`, `FIELD`, `TYPE` |
| `@Documented` | Include in Javadoc | (no value) |
| `@Inherited` | Subclass inheritance | (no value) |
| `@Repeatable` | Allow multiple applications | Container annotation |

#### Syntax Rules

- Declare with `@interface` and a name.
- Apply `@Retention(RUNTIME)` if the annotation must be read reflectively.
- Apply `@Target(...)` to restrict where the annotation can be placed.
- Elements are declared as methods with no parameters.
- Default values make elements optional.
- Use `@Repeatable` to allow multiple instances of the same annotation.
- Use `@Inherited` to allow subclasses to inherit the annotation from superclasses.

#### Constraints and Limitations

- Custom annotations without `@Retention(RUNTIME)` cannot be read via reflection.
- `@Target` restriction causes compile-time errors if violated.
- `@Inherited` only works for class-level annotations, not methods or fields.
- `@Repeatable` requires a container annotation.
- Annotations cannot be used to change program semantics without a processor or reflective code.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Custom Annotation for Runtime Processing

**Setup Guide**: Save as `CustomAnnotationDemo.java`, compile with `javac`, and run with `java`.

```java
// CustomAnnotationDemo.java
import java.lang.annotation.*;
import java.lang.reflect.*;

// Custom annotation with RUNTIME retention
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Loggable {
    String value() default "default message";
    int level() default 1;
}

class Service {
    @Loggable(value = "Processing order", level = 2)
    public void processOrder(String orderId) {
        System.out.println("Processing: " + orderId);
    }

    @Loggable
    public void cancelOrder(String orderId) {
        System.out.println("Cancelling: " + orderId);
    }

    public void internalMethod() {
        System.out.println("Internal");
    }
}

public class CustomAnnotationDemo {
    public static void main(String[] args) throws Exception {
        System.out.println("=== Custom Annotation Demo ===\n");

        Service service = new Service();
        Class<?> clazz = service.getClass();

        // Process all methods with @Loggable
        for (Method method : clazz.getDeclaredMethods()) {
            Loggable loggable = method.getAnnotation(Loggable.class);
            if (loggable != null) {
                System.out.println("Method: " + method.getName());
                System.out.println("  Message: " + loggable.value());
                System.out.println("  Level: " + loggable.level());
                System.out.println("  Invoking...");
                // Simulate invocation (simplified)
            }
        }

        // Direct invocation
        service.processOrder("ORD-001");
        service.cancelOrder("ORD-002");
    }
}
```

**Expected Output**:
```
=== Custom Annotation Demo ===

Method: processOrder
  Message: Processing order
  Level: 2
  Invoking...
Method: cancelOrder
  Message: default message
  Level: 1
  Invoking...

Processing: ORD-001
Cancelling: ORD-002
```

**Why This Output**: The `@Loggable` annotation is declared with `RUNTIME` retention and `METHOD` target. At runtime, the program iterates over methods, retrieves the annotation via `getAnnotation()`, and reads its elements. `processOrder` uses explicit values; `cancelOrder` uses defaults. `internalMethod` has no annotation and is skipped.

---

### Real-World Cases

- **Spring**: `@Autowired`, `@Component`, `@RequestMapping` configure the Spring container.
- **Hibernate Validator**: `@NotNull`, `@Size`, `@Email` validate bean properties.
- **Jackson**: `@JsonProperty`, `@JsonIgnore`, `@JsonFormat` customize JSON mapping.
- **Lombok**: `@Data`, `@Builder`, `@Getter` generate boilerplate code via annotation processing.
- **MapStruct**: `@Mapper`, `@Mapping` generate type-safe bean mapping code.

### References

- Annotations - Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/annotations/
- Creating Custom Annotations - Dev.java - https://dev.java/learn/annotations/

---

## Core Concept 4: Retention Policies

### Definitions

**Core Definition**: Retention policy controls the lifecycle of an annotation, determining whether it is discarded by the compiler (SOURCE), stored in the class file but not retained at runtime (CLASS), or retained by the VM at runtime for reflective access (RUNTIME).

**Technical Definition**: The `@Retention` meta-annotation specifies the retention policy of an annotation type. The `RetentionPolicy` enum has three constants: `SOURCE` (annotations are discarded by the compiler), `CLASS` (annotations are recorded in the class file but not retained by the VM at runtime; this is the default), and `RUNTIME` (annotations are recorded in the class file and retained by the VM at runtime, so they may be read reflectively).

**Beginner-Friendly Explanation**: Retention policy is like the lifespan of a sticky note. `SOURCE` notes are thrown away as soon as the code is compiled—they're just for the compiler. `CLASS` notes stay on the compiled code but the runtime ignores them. `RUNTIME` notes stay on the compiled code and the runtime can read them—these are the ones used by frameworks.

### Purposes

- To control whether annotations are available at compile time only, in bytecode, or at runtime.
- To minimize bytecode size by discarding SOURCE annotations.
- To enable runtime reflection for framework processing with RUNTIME.
- To support compile-time-only processing (e.g., `@Override`) with SOURCE.
- To provide a default (CLASS) for annotations that don't need runtime access.
- To balance between metadata availability and runtime overhead.

### Syntax Rules and Structure

#### Complete General Syntax: Retention Policies

```
RETENTION POLICY SYNTAX
│
├── @Retention(RetentionPolicy.SOURCE)
│   ├── Discarded by compiler
│   ├── Not in class file
│   ├── Not available at runtime
│   └── Example: @Override, @SuppressWarnings
│
├── @Retention(RetentionPolicy.CLASS)
│   ├── Recorded in class file
│   ├── Not retained by VM at runtime
│   ├── Default policy if @Retention absent
│   └── Example: Some bytecode analysis annotations
│
└── @Retention(RetentionPolicy.RUNTIME)
    ├── Recorded in class file
    ├── Retained by VM at runtime
    ├── Readable via reflection
    └── Example: @Deprecated, custom framework annotations
```

#### Component Breakdown

| Policy | Class File | Runtime | Reflection | Typical Use |
|--------|-----------|---------|------------|-------------|
| `SOURCE` | No | No | No | Compiler checks |
| `CLASS` | Yes | No | No | Bytecode tools |
| `RUNTIME` | Yes | Yes | Yes | Frameworks |

#### Syntax Rules

- `@Retention` is applied to the annotation declaration, not to its usage.
- The value is a single `RetentionPolicy` constant.
- If `@Retention` is omitted, the default is `CLASS`.
- `RUNTIME` is required for annotations read via `getAnnotation()`.
- `SOURCE` annotations are the least expensive (no bytecode overhead).
- `CLASS` annotations are visible to bytecode analysis tools but not to reflection.

#### Constraints and Limitations

- SOURCE annotations cannot be processed at runtime.
- CLASS annotations cannot be read via reflection (a common source of confusion).
- RUNTIME annotations increase class file size and runtime metadata.
- The default (CLASS) is often not what developers expect; always specify `@Retention` explicitly.
- Retention policy cannot be changed after the annotation is compiled.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Comparing Retention Policies

**Setup Guide**: Save as `RetentionDemo.java`, compile with `javac`, and run with `java`.

```java
// RetentionDemo.java
import java.lang.annotation.*;
import java.lang.reflect.*;

// SOURCE retention — discarded by compiler
@Retention(RetentionPolicy.SOURCE)
@interface SourceAnnotation {
    String value();
}

// CLASS retention — in class file, not runtime-visible
@Retention(RetentionPolicy.CLASS)
@interface ClassAnnotation {
    String value();
}

// RUNTIME retention — visible via reflection
@Retention(RetentionPolicy.RUNTIME)
@interface RuntimeAnnotation {
    String value();
}

@SourceAnnotation("source")
@ClassAnnotation("class")
@RuntimeAnnotation("runtime")
public class RetentionDemo {
    public static void main(String[] args) throws Exception {
        System.out.println("=== Retention Policy Demo ===\n");

        Class<?> clazz = RetentionDemo.class;

        // Check each annotation's runtime availability
        System.out.println("SOURCE annotation present: " +
            (clazz.getAnnotation(SourceAnnotation.class) != null));
        System.out.println("CLASS annotation present: " +
            (clazz.getAnnotation(ClassAnnotation.class) != null));
        System.out.println("RUNTIME annotation present: " +
            (clazz.getAnnotation(RuntimeAnnotation.class) != null));

        System.out.println("\nOnly RUNTIME annotations are visible at runtime.");
        System.out.println("SOURCE and CLASS annotations are not retained by the VM.");
    }
}
```

**Expected Output**:
```
=== Retention Policy Demo ===

SOURCE annotation present: false
CLASS annotation present: false
RUNTIME annotation present: true

Only RUNTIME annotations are visible at runtime.
SOURCE and CLASS annotations are not retained by the VM.
```

**Why This Output**: The program applies three annotations with different retention policies to `RetentionDemo`. At runtime, `getAnnotation()` returns `null` for SOURCE (discarded by compiler) and CLASS (not retained by VM), but returns the annotation for RUNTIME. This demonstrates that only `RUNTIME` annotations are accessible via reflection.

---

### Real-World Cases

- **`@Override`**: SOURCE retention; compiler-only check, no runtime overhead.
- **`@SuppressWarnings`**: SOURCE retention; compiler-only.
- **Jackson `@JsonProperty`**: RUNTIME retention; Jackson reads it reflectively.
- **Hibernate `@Entity`**: RUNTIME retention; Hibernate reads it to map classes to tables.
- **Lombok `@Data`**: SOURCE retention; Lombok's annotation processor runs at compile time.
- **Bytecode Analysis Tools**: CLASS retention annotations are visible to ASM and other bytecode libraries.

### References

- RetentionPolicy - Java SE 8 - https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/RetentionPolicy.html
- @Retention - Java SE API - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/annotation/Retention.html

---

## Core Concept 5: Targets

### Definitions

**Core Definition**: Annotation targets restrict the syntactic locations where an annotation can be applied, using `@Target` with `ElementType` constants such as `TYPE`, `METHOD`, `FIELD`, `PARAMETER`, or `TYPE_USE`.

**Technical Definition**: The `@Target` meta-annotation restricts the contexts in which an annotation type is applicable. The `ElementType` enum provides constants for declaration contexts (`ANNOTATION_TYPE`, `CONSTRUCTOR`, `FIELD`, `LOCAL_VARIABLE`, `METHOD`, `PACKAGE`, `MODULE`, `PARAMETER`, `TYPE`) and type contexts (`TYPE_USE`, `TYPE_PARAMETER`). The `TYPE_USE` constant corresponds to the 15 type contexts in JLS 4.11, as well as to two declaration contexts: type declarations and type parameter declarations. This enables type annotations for static analysis frameworks like the Checker Framework.

**Beginner-Friendly Explanation**: Target restrictions are like rules about where you can stick your sticky notes. Some notes can only go on methods (`@Target(METHOD)`), some only on classes (`@Target(TYPE)`), and some can go almost anywhere, including on type usages like `List<@NonNull String>` (`@Target(TYPE_USE)`). If you try to put a sticky note in the wrong place, the compiler will complain.

### Purposes

- To enforce correct annotation placement at compile time.
- To document the intended usage of an annotation.
- To prevent misuse of annotations by other developers.
- To enable type annotations for static analysis (Checker Framework).
- To support fine-grained placement restrictions for framework design.
- To allow annotations on type parameters and type usages.

### Syntax Rules and Structure

#### Complete General Syntax: @Target ElementTypes

```
@TARGET ELEMENT TYPES
│
├── Declaration Contexts
│   ├── ANNOTATION_TYPE — annotation type declaration
│   ├── CONSTRUCTOR — constructor declaration
│   ├── FIELD — field declaration
│   ├── LOCAL_VARIABLE — local variable declaration
│   ├── METHOD — method declaration
│   ├── PACKAGE — package declaration
│   ├── MODULE — module declaration
│   ├── PARAMETER — formal parameter declaration
│   └── TYPE — class, interface, enum, or record declaration
│
├── Type Contexts
│   ├── TYPE_USE — any type usage (e.g., List<@NonNull String>)
│   └── TYPE_PARAMETER — type parameter declaration (e.g., <@NonNull T>)
│
└── Multiple Targets
    └── @Target({METHOD, FIELD, TYPE})
```

#### Component Breakdown

| ElementType | Applies To | Example |
|-------------|-----------|---------|
| `TYPE` | Class, interface, enum, record | `@Entity class User` |
| `METHOD` | Method | `@Test void test()` |
| `FIELD` | Field | `@Column String name` |
| `PARAMETER` | Method parameter | `void set(@NotNull String x)` |
| `CONSTRUCTOR` | Constructor | `@Inject public Service()` |
| `TYPE_USE` | Type usage | `List<@NonNull String>` |
| `TYPE_PARAMETER` | Type parameter | `class Box<@NonNull T>` |

#### Syntax Rules

- `@Target` is applied to the annotation declaration.
- The value is one or more `ElementType` constants, comma-separated in braces.
- If `@Target` is omitted, the annotation can be applied almost anywhere (except type contexts).
- `TYPE_USE` includes type declarations and type parameter declarations for convenience.
- `TYPE_USE` annotations are used by static analysis frameworks (e.g., Checker Framework).
- Multiple targets allow an annotation to be used in several contexts.

#### Constraints and Limitations

- Incorrect placement causes a compile-time error.
- `TYPE_USE` annotations can appear in complex type positions (e.g., arrays, generics, casts).
- `@Target` does not prevent runtime misuse if reflection is used.
- The default (no `@Target`) allows broader placement than most annotations need.
- `TYPE_USE` requires Java 8+.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Target-Restricted Annotations

**Setup Guide**: Save as `TargetDemo.java`, compile with `javac`, and run with `java`.

```java
// TargetDemo.java
import java.lang.annotation.*;
import java.lang.reflect.*;

// Annotation restricted to methods
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface MethodOnly {
    String value() default "";
}

// Annotation restricted to fields
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface FieldOnly {
    String value() default "";
}

// Annotation restricted to type usage
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_USE)
@interface NonNull {}

public class TargetDemo {
    @FieldOnly("name field")
    private String name;

    @MethodOnly("process method")
    public void process() {}

    // TYPE_USE annotation on a generic type
    private java.util.List<@NonNull String> safeList;

    public static void main(String[] args) throws Exception {
        System.out.println("=== Target Restriction Demo ===\n");

        Class<?> clazz = TargetDemo.class;

        // Check field annotation
        Field nameField = clazz.getDeclaredField("name");
        FieldOnly fieldAnn = nameField.getAnnotation(FieldOnly.class);
        System.out.println("Field annotation: " + fieldAnn.value());

        // Check method annotation
        Method processMethod = clazz.getMethod("process");
        MethodOnly methodAnn = processMethod.getAnnotation(MethodOnly.class);
        System.out.println("Method annotation: " + methodAnn.value());

        // TYPE_USE annotation on field type
        Field listField = clazz.getDeclaredField("safeList");
        AnnotatedType annotatedType = listField.getAnnotatedType();
        System.out.println("Field type: " + annotatedType.getType());
        System.out.println("Type annotations: " +
            annotatedType.getAnnotations().length);

        System.out.println("\nAll annotations placed correctly per @Target rules.");
    }
}
```

**Expected Output**:
```
=== Target Restriction Demo ===

Field annotation: name field
Method annotation: process method
Field type: java.util.List<java.lang.String>
Type annotations: 0

All annotations placed correctly per @Target rules.
```

**Why This Output**: The `@FieldOnly` annotation can only be placed on fields; `@MethodOnly` only on methods. The `@NonNull` annotation with `TYPE_USE` target is placed on the generic type argument `@NonNull String` inside `List<@NonNull String>`. The program reads the field and method annotations directly. The `TYPE_USE` annotation on the generic type argument is not directly visible via `getAnnotatedType()` in this simplified example (it would require a more complex type annotation query), but the placement compiles correctly.

---

### Real-World Cases

- **JPA `@Column`**: `@Target({METHOD, FIELD})` — can be placed on getter or field.
- **Spring `@Autowired`**: `@Target({CONSTRUCTOR, METHOD, PARAMETER, FIELD, ANNOTATION_TYPE})` — flexible placement.
- **JUnit `@Test`**: `@Target({METHOD, ANNOTATION_TYPE})` — method only.
- **Checker Framework `@NonNull`**: `@Target(TYPE_USE)` — type annotations for static analysis.
- **Lombok `@Getter`**: `@Target({FIELD, TYPE})` — on fields or classes.

### References

- ElementType - Java SE 11 - https://cr.openjdk.org/~iris/se/11/pr/java-se-11-pr-spec-02/api/java.base/java/lang/annotation/ElementType.html
- @Target - Java SE API - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/annotation/Target.html

---

## Core Concept 6: Annotation Processing

### Definitions

**Core Definition**: Annotation processing is a compile-time mechanism provided by the Pluggable Annotation Processing API (JSR 269) that allows processors to inspect annotations, generate source files, validate schemas, and emit diagnostics during compilation.

**Technical Definition**: The Pluggable Annotation Processing API is defined in the `javax.annotation.processing` package. Annotation processors implement the `Processor` interface (or extend `AbstractProcessor`) and are discovered via the `META-INF/services/javax.annotation.processing.Processor` service file. During compilation, the compiler invokes processors in rounds. Processors receive a `RoundEnvironment` that provides access to annotated elements, and a `Filer` for creating new source files, class files, or resources. The `Messager` interface allows processors to report errors, warnings, and notes. Processors can be configured with `@SupportedAnnotationTypes`, `@SupportedSourceVersion`, and `@SupportedOptions`.

**Beginner-Friendly Explanation**: Annotation processing is like having a helper who reads your sticky notes while you're writing code and generates additional code based on them. For example, if you mark a class with `@JsonSerializable`, a processor can automatically generate a JSON serializer for it—all before your code is even compiled. This is how Lombok, MapStruct, and Dagger work.

### Purposes

- To generate source code automatically based on annotations.
- To validate annotation usage at compile time.
- To emit compiler errors and warnings for invalid annotations.
- To generate configuration files, metadata, or resources.
- To reduce boilerplate code through code generation.
- To enforce architectural or schema constraints during compilation.

### Syntax Rules and Structure

#### Complete General Syntax: Annotation Processor

```
ANNOTATION PROCESSOR SYNTAX
│
├── 1. Define the Processor
│   ├── @SupportedAnnotationTypes("com.example.MyAnnotation")
│   ├── @SupportedSourceVersion(SourceVersion.RELEASE_21)
│   └── public class MyProcessor extends AbstractProcessor {
│           @Override
│           public boolean process(Set<? extends TypeElement> annotations,
│                                  RoundEnvironment roundEnv) {
│               // Process annotations
│               return true;
│           }
│       }
│
├── 2. Register the Processor
│   └── META-INF/services/javax.annotation.processing.Processor
│       └── com.example.MyProcessor
│
└── 3. Generate Code
    ├── Filer filer = processingEnv.getFiler();
    ├── JavaFileObject file = filer.createSourceFile("GeneratedClass");
    └── Writer writer = file.openWriter();
```

#### Component Breakdown

| Component | Interface/Class | Purpose |
|-----------|----------------|---------|
| Processor | `Processor` / `AbstractProcessor` | Entry point for processing |
| RoundEnvironment | `RoundEnvironment` | Access to annotated elements |
| Filer | `Filer` | Create new files |
| Messager | `Messager` | Report diagnostics |
| ProcessingEnvironment | `ProcessingEnvironment` | Access to tools and options |

#### Syntax Rules

- Extend `AbstractProcessor` for convenience.
- Annotate with `@SupportedAnnotationTypes` to declare which annotations the processor handles.
- Annotate with `@SupportedSourceVersion` to declare the supported Java version.
- Register the processor in `META-INF/services/javax.annotation.processing.Processor`.
- Use `Filer.createSourceFile()` to generate new Java source files.
- Use `Messager.printMessage()` to report errors, warnings, or notes.
- Return `true` from `process()` if the annotations are claimed by this processor.

#### Constraints and Limitations

- Processors run during compilation; they cannot modify existing source files.
- Generated files are compiled in subsequent rounds.
- Processors must be registered via the service file to be discovered.
- Overlapping processor claims can cause conflicts.
- Annotation processing adds compile-time overhead.
- `null` is not a valid annotation element value.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Simple Annotation Processor

**Setup Guide**: This example requires three files: an annotation, a processor, and a service registration. Compile the annotation and processor, then compile a class using the annotation with the processor on the classpath.

**File 1: `HelloAnnotation.java`**
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
public @interface HelloAnnotation {
    String value() default "World";
}
```

**File 2: `HelloProcessor.java`**
```java
import javax.annotation.processing.*;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.*;
import java.io.Writer;
import java.util.Set;

@SupportedAnnotationTypes("HelloAnnotation")
@SupportedSourceVersion(SourceVersion.RELEASE_21)
public class HelloProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations,
                          RoundEnvironment roundEnv) {
        for (TypeElement annotation : annotations) {
            for (Element element : roundEnv.getElementsAnnotatedWith(annotation)) {
                String className = element.getSimpleName().toString();
                String greeting = element.getAnnotation(HelloAnnotation.class).value();

                try {
                    // Generate a new source file
                    String generatedName = className + "Generated";
                    Writer writer = processingEnv.getFiler()
                        .createSourceFile(generatedName)
                        .openWriter();

                    writer.write("public class " + generatedName + " {\n");
                    writer.write("    public static void main(String[] args) {\n");
                    writer.write("        System.out.println(\"" + greeting + "\");\n");
                    writer.write("    }\n");
                    writer.write("}\n");
                    writer.close();

                    processingEnv.getMessager().printMessage(
                        Diagnostic.Kind.NOTE,
                        "Generated " + generatedName + ".java");

                } catch (Exception e) {
                    processingEnv.getMessager().printMessage(
                        Diagnostic.Kind.ERROR,
                        "Failed to generate: " + e.getMessage());
                }
            }
        }
        return true;
    }
}
```

**File 3: `META-INF/services/javax.annotation.processing.Processor`**
```
HelloProcessor
```

**File 4: `Greeter.java`** (to be compiled with the processor)
```java
@HelloAnnotation("Hello from Annotation Processor!")
public class Greeter {
    // Annotation processor will generate GreeterGenerated.java
}
```

**Compilation Commands**:
```bash
# Compile annotation and processor
javac HelloAnnotation.java HelloProcessor.java

# Compile Greeter with processor on classpath
javac -cp . Greeter.java

# Run the generated class
java GreeterGenerated
```

**Expected Output**:
```
Hello from Annotation Processor!
```

**Why This Output**: The `HelloProcessor` is registered via the service file and is invoked during compilation of `Greeter`. It reads the `@HelloAnnotation` value and generates `GreeterGenerated.java` with a `main` method that prints the greeting. The generated file is compiled in a subsequent round, and running `GreeterGenerated` produces the output. This demonstrates the core annotation processing workflow: discover annotation, generate source, compile generated source.

---

### Real-World Cases

- **Lombok**: Generates getters, setters, `equals`, `hashCode`, and `toString` from `@Data`.
- **MapStruct**: Generates type-safe bean mapping code from `@Mapper` interfaces.
- **Dagger**: Generates dependency injection code from `@Inject` and `@Component`.
- **Room**: Generates database access code from `@Entity` and `@Dao` in Android.
- **Immutables**: Generates immutable value objects from `@Value.Immutable`.

### References

- Package javax.annotation.processing - Java SE 8 - https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/package-summary.html
- JSR 269: Pluggable Annotation Processing API - https://www.jcp.org/en/jsr/detail?id=269
- Annotation Processing - Dev.java - https://dev.java/learn/annotations/

---

## Deprecation and Safety Notes

| Feature | Status | Notes |
|---------|--------|-------|
| `@Deprecated` (no elements) | Active | Use `since` and `forRemoval` since Java 9. |
| `Class.newInstance()` | Deprecated | Use `Constructor.newInstance()` instead. |
| `finalize()` | Deprecated (Java 9+) | Use `Cleaner`. |
| `@Serial` | Active (Java 14+) | Marks serialization-related members. |
| `@SafeVarargs` on non-final methods | Not allowed | Only `static`, `final`, or `private` since Java 9. |
| `--permit-illegal-access` | Removed (Java 10+) | Use `--add-opens`. |
| `sun.misc.Unsafe` memory access | Deprecated (JDK 23) | Use `VarHandle` or FFM API. |

---

## References

### Official Specifications and Documentation

- Chapter 9. Interfaces: Annotation Interfaces - Java Language Specification, SE 17 - https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html#jls-9.6
- Predefined Annotation Types - Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/annotations/predefined.html
- RetentionPolicy - Java SE 8 - https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/RetentionPolicy.html
- ElementType - Java SE 11 - https://cr.openjdk.org/~iris/se/11/pr/java-se-11-pr-spec-02/api/java.base/java/lang/annotation/ElementType.html
- Package javax.annotation.processing - Java SE 8 - https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/package-summary.html
- Annotation Interface Serial - Java SE 21 - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Serial.html
- Deprecated - Java SE 9 - https://docs.oracle.com/javase/9/docs/api/java/lang/Deprecated.html
- @Target - Java SE API - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/annotation/Target.html

### JSR Specifications

- JSR 269: Pluggable Annotation Processing API - https://www.jcp.org/en/jsr/detail?id=269
- JSR 175: A Metadata Facility for the Java Programming Language - https://www.jcp.org/en/jsr/detail?id=175

### Tutorials and Guides

- Annotations - Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/annotations/
- Creating Custom Annotations - Dev.java - https://dev.java/learn/annotations/
- Annotation Processing - Dev.java - https://dev.java/learn/annotations/
- Annotation Type SafeVarargs - Java SE 21 - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/SafeVarargs.html