# Java Serialization Concepts: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Java Serialization** is a built-in framework that converts the state of an object into a byte stream, enabling the object to be persisted to disk or transmitted over a network. **Deserialization** is the reverse process—reconstructing the object from its byte representation.

### Technical Definition

The Java Serialization framework enables object state to be frozen, stored, and unfrozen again into objects . An object is serializable if its class or any of its superclasses implements the `java.io.Serializable` marker interface or its subinterface `Externalizable` . The framework uses `ObjectOutputStream` for writing objects and `ObjectInputStream` for reading them . Serialization captures all fields of an object, including private fields, unless they are marked `transient` .

### Beginner-Friendly Explanation

Imagine you have a complex LEGO structure (an object). To save it for later, you carefully disassemble it and write down instructions for rebuilding it (serialization). Later, you follow those instructions to rebuild the exact same LEGO structure (deserialization). The instructions (byte stream) can be stored in a file or sent to a friend over the network.

### Key Characteristics

- **Marker interface**: `Serializable` has no methods; it simply tags a class as eligible for serialization .
- **Automatic field capture**: The default mechanism serializes all non-transient fields, including private ones .
- **Version tracking**: `serialVersionUID` ensures class compatibility during deserialization .
- **Selective exclusion**: The `transient` keyword prevents specific fields from being serialized .
- **Security risks**: Deserializing untrusted data is inherently dangerous and can lead to remote code execution .

### Prerequisites

- Basic understanding of classes, fields, and object creation.
- Familiarity with I/O streams (`ObjectInputStream`, `ObjectOutputStream`).
- Knowledge of the `try-with-resources` statement.

### Related Programming Areas

- **Remote Method Invocation (RMI)**: Serialization enables method calls over a network .
- **JavaBeans**: Serialization is used for bean persistence .
- **Caching and Session Replication**: Objects are serialized to shared storage.
- **Modern Data Interchange**: JSON, XML, and Protocol Buffers as alternatives .

### Core Concepts / Features

1. Object Serialization and Deserialization Mechanisms
2. The `Serializable` Marker Interface
3. The `transient` Keyword
4. Serialization Risks, Vulnerabilities, and Security Best Practices
5. Versioning Control with `serialVersionUID`
6. Modern Alternatives and Object Filters


## Core Concept 1: Object Serialization and Deserialization Mechanisms

### Definitions

**Core Definition**: Serialization writes an object's state to a byte stream; deserialization reads that byte stream and reconstructs the object.

**Technical Definition**: `ObjectOutputStream.writeObject(Object)` serializes an object, and `ObjectInputStream.readObject()` deserializes it . When a `Serializable` class does not implement `writeObject()` or `readObject()`, the default mechanism serializes all non-transient fields, including non-public ones .

**Beginner-Friendly Explanation**: It's like saving a video game. The game (object) saves its current state (fields) to a file. When you load the game, it reads the file and restores everything exactly as it was.

### Purposes

- To persist object state across program executions.
- To transmit objects over a network (e.g., RMI).
- To cache objects for later retrieval.
- To enable session replication in clustered applications.

### Syntax Rules and Structure

**Serialization:**
```java
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("object.ser"))) {
    oos.writeObject(myObject);
}
```

**Deserialization:**
```java
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("object.ser"))) {
    MyClass obj = (MyClass) ois.readObject();
}
```

**Syntax Rules:**
- The class must implement `Serializable`.
- All fields that are objects must also be serializable (or `transient`).
- Use `try-with-resources` to ensure streams are closed.

**Constraints and Limitations:**
- Static fields are not serialized.
- Serialization captures the object graph; circular references are handled via object handles .
- A deserialized object is a **copy**, not the same instance.

### Annotated Complete Code Examples

**Example 1: Basic Serialization and Deserialization**

```java
import java.io.*;

class Person implements Serializable {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}

public class SerializationDemo {
    public static void main(String[] args) throws Exception {
        Person p = new Person("Alice", 30);

        // Serialize
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("person.ser"))) {
            oos.writeObject(p);
            System.out.println("Serialized: " + p);
        }

        // Deserialize
        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream("person.ser"))) {
            Person restored = (Person) ois.readObject();
            System.out.println("Deserialized: " + restored);
        }
    }
}
```

**Expected Output:**
```
Serialized: Person{name='Alice', age=30}
Deserialized: Person{name='Alice', age=30}
```

**Why This Output Occurs:** The `Person` object is written to `person.ser` and read back. The restored object is a new instance with the same field values .

### Real-World Cases

- **Session management**: Web servers serialize HTTP session objects.
- **RMI**: Method arguments and return values are serialized .
- **Caching**: Objects are serialized to disk for later retrieval.

### References

- SEI CERT – Serialization (Libraries) - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1529-19/Libraries_7bb97f4cbd5847e58953970f6dd21802-201124-1529-20.pdf
- Oracle – Addressing Serialization Vulnerabilities - https://docs.oracle.com/en/java/javase/21/core/addressing-serialization-vulnerabilities.html


## Core Concept 2: The `Serializable` Marker Interface

### Definitions

**Core Definition**: `java.io.Serializable` is an empty marker interface that tags a class as eligible for Java's built-in serialization mechanism.

**Technical Definition**: `Serializable` is an empty interface with no methods . Implementing it is enough to make most classes serializable . For fine-grained control, a class can implement `writeObject()` and `readObject()` with exact signatures .

**Beginner-Friendly Explanation**: It's like a stamp that says "this object is allowed to be saved." Without the stamp, the serialization system refuses to process it.

### Purposes

- To declare that a class's instances can be converted to byte streams.
- To enable automatic serialization without writing any methods.
- To serve as a marker for frameworks (e.g., RMI, JavaBeans) .

### Syntax Rules and Structure

```java
public class MyClass implements Serializable {
    // fields and methods
}
```

**Optional custom methods:**
```java
private void writeObject(ObjectOutputStream out) throws IOException { }
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException { }
```

**Syntax Rules:**
- The signatures must match exactly .
- `Serializable` has no methods to implement.
- The marker is inherited by subclasses.

**Constraints and Limitations:**
- The marker is "problematic" because method implementation is optional and not enforced by the compiler .
- All non-transient fields must be serializable.

### Annotated Complete Code Examples

**Example 1: Custom writeObject/readObject**

```java
import java.io.*;

class SecureData implements Serializable {
    private String username;
    private transient String password; // Not serialized

    public SecureData(String username, String password) {
        this.username = username;
        this.password = password;
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject(); // Serialize non-transient fields
        out.writeObject("ENCRYPTED:" + password); // Custom handling
    }

    private void readObject(ObjectInputStream in) 
            throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        String encrypted = (String) in.readObject();
        this.password = encrypted.replace("ENCRYPTED:", "");
    }

    @Override
    public String toString() {
        return "SecureData{username='" + username + "', password='" + password + "'}";
    }
}
```

**Expected Output (if deserialized):** `SecureData{username='alice', password='secret'}`

**Why This Output Occurs:** The custom `writeObject` encrypts the password before writing; `readObject` decrypts it .

### Real-World Cases

- **Custom serialization**: Encrypting sensitive fields or validating invariants.
- **Backward compatibility**: Handling old serialized formats.

### References

- OpenJDK – Serializable (Java 9+) - https://cr.openjdk.org/~jboes/webrevs/8231186/webrev.02/src/java.base/share/classes/java/io/Serializable.java.sdiff.html
- SEI CERT – Serialization (Custom Methods) - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1529-19/Libraries_7bb97f4cbd5847e58953970f6dd21802-201124-1529-20.pdf


## Core Concept 3: The `transient` Keyword

### Definitions

**Core Definition**: The `transient` modifier marks a field as excluded from serialization, preventing its state from being written to or restored from the byte stream.

**Technical Definition**: Transient fields are not persistent and will not be saved by any persistence mechanism . Marking a field as `transient` prevents its state from appearing in the stream and from being restored during deserialization .

**Beginner-Friendly Explanation**: It's like having a private diary that you don't photocopy when saving your room's contents. The diary stays with you; only everything else is saved.

### Purposes

- To exclude sensitive data (passwords, keys) from serialized streams .
- To exclude non-serializable resources (file handles, sockets) .
- To exclude derived or cached values that can be recomputed.

### Syntax Rules and Structure

```java
private transient String sensitiveData;
private transient FileInputStream fileHandle;
```

**Syntax Rules:**
- `transient` can be applied to fields only.
- Transient fields are skipped by default serialization.
- After deserialization, transient fields have their default values (`null`, `0`, `false`).

**Constraints and Limitations:**
- If a class implements custom `writeObject`/`readObject`, it can choose to serialize transient fields manually.
- Transient does not affect static fields (which are never serialized).

### Annotated Complete Code Examples

**Example 1: Protecting Sensitive Data**

```java
import java.io.*;

class User implements Serializable {
    private String username;
    private transient String password; // Never serialized

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{username='" + username + "', password='" + password + "'}";
    }
}

public class TransientDemo {
    public static void main(String[] args) throws Exception {
        User user = new User("alice", "secret123");

        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("user.ser"))) {
            oos.writeObject(user);
        }

        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream("user.ser"))) {
            User restored = (User) ois.readObject();
            System.out.println("Original: " + user);
            System.out.println("Restored: " + restored);
        }
    }
}
```

**Expected Output:**
```
Original: User{username='alice', password='secret123'}
Restored: User{username='alice', password='null'}
```

**Why This Output Occurs:** The `password` field is `transient`, so it is not written to the stream. After deserialization, it has its default value `null` .

### Real-World Cases

- **Password fields**: Never serialize credentials.
- **File handles**: Direct handles to system resources should not be serialized .
- **Cached objects**: Recomputable values should be transient.

### References

- OpenJDK – Serialization Architecture (Transient) - https://cr.openjdk.org/~ihse/JDK-8179556-add-legal-copyright-html/webrev.01/jdk/src/java.base/share/specs/serialization/serial-arch.md.sdiff.html
- Princeton – Security in Object Serialization - https://www.cs.princeton.edu/courses/archive/fall97/cs461/jdkdocs/guide/serialization/spec/security.doc.html


## Core Concept 4: Serialization Risks, Vulnerabilities, and Security Best Practices

### Definitions

**Core Definition**: Deserialization of untrusted data is inherently dangerous because the byte stream controls which classes are instantiated, what values their fields receive, and what methods are invoked.

**Technical Definition**: "Gadget classes" can be deserialized maliciously to cause denial of service or remote code execution . Deserialization is code execution because the `readObject` method can contain custom code .

**Beginner-Friendly Explanation**: Imagine opening a package from a stranger. Inside, there's a bomb disguised as a gift. Deserialization is like automatically opening and activating whatever is in the package. If the package is from an untrusted source, it could destroy your system.

### Purposes (of the security measures)

- To prevent remote code execution (RCE) attacks.
- To prevent denial of service (DoS) attacks.
- To protect application integrity and data confidentiality.

### Syntax Rules and Structure (Best Practices)

1. **Do not deserialize untrusted data** .
2. **Use serialization filters** to allow or reject classes .
3. **Use SSL** to encrypt and authenticate connections .
4. **Validate field values** in `readObject` .
5. **Mark sensitive fields as `transient`** .
6. **Use `ObjectInputFilter`** to control deserialization .

**Example filter:**
```java
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "example.*;java.base/*;!*"
);
ObjectInputStream ois = new ObjectInputStream(inputStream);
ois.setObjectInputFilter(filter);
```

**Constraints and Limitations:**
- Filters are **not enabled by default** .
- Filters can be bypassed if not configured correctly.
- The default serialization mechanism is "problematic" due to its complexity and security issues .

### Annotated Complete Code Examples

**Example 1: Using ObjectInputFilter**

```java
import java.io.*;

class SafeClass implements Serializable {
    private String data;
    public SafeClass(String data) { this.data = data; }
    @Override public String toString() { return "SafeClass{data='" + data + "'}"; }
}

public class FilterDemo {
    public static void main(String[] args) throws Exception {
        // Serialize a safe object
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("safe.ser"))) {
            oos.writeObject(new SafeClass("Hello"));
        }

        // Deserialize with a filter that only allows SafeClass
        ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
            "FilterDemo$SafeClass;!*"  // Allow SafeClass, reject everything else
        );

        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream("safe.ser"))) {
            ois.setObjectInputFilter(filter);
            SafeClass obj = (SafeClass) ois.readObject();
            System.out.println("Deserialized: " + obj);
        } catch (InvalidClassException e) {
            System.out.println("Rejected by filter: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Deserialized: SafeClass{data='Hello'}
```

**Why This Output Occurs:** The filter allows `SafeClass` to be deserialized while rejecting all other classes .

### Real-World Cases

- **Apache Commons Collections vulnerability**: A well-known gadget chain.
- **Spring Framework**: Uses `ObjectInputFilter` for RMI protection .
- **Financial systems**: Never deserialize untrusted data.

### References

- Oracle – Addressing Serialization Vulnerabilities - https://docs.oracle.com/en/java/javase/21/core/addressing-serialization-vulnerabilities.html
- OpenJDK – ObjectInputFilter (Java 17) - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/ObjectInputFilter.html
- Oracle – Serialization Filtering - https://docs.oracle.com/javase/8/docs/technotes/guides/serialization/filters/serialization-filtering.html


## Core Concept 5: Versioning Control with `serialVersionUID`

### Definitions

**Core Definition**: `serialVersionUID` is a unique identifier associated with each serializable class, used during deserialization to verify that the sender and receiver have loaded compatible classes.

**Technical Definition**: The serialization runtime associates with each serializable class a version number called `serialVersionUID`. If the receiver's class has a different `serialVersionUID` than the sender's, deserialization results in `InvalidClassException` . It is **strongly recommended** that all serializable classes explicitly declare `serialVersionUID` .

**Beginner-Friendly Explanation**: It's like a version number on a software update. If you try to open a file created by version 2 with version 1, you might get errors. `serialVersionUID` ensures that the class reading the object is compatible with the class that wrote it.

### Purposes

- To verify class compatibility during deserialization.
- To prevent `InvalidClassException` when class definitions change.
- To enable controlled evolution of serializable classes.

### Syntax Rules and Structure

```java
private static final long serialVersionUID = 1L;
```

**Syntax Rules:**
- Must be `static`, `final`, and of type `long` .
- Can have any access modifier, but `private` is recommended .
- If not declared, the JVM computes a default based on class details .
- The default computation is **highly sensitive** to compiler implementations .

**Constraints and Limitations:**
- Array classes cannot declare an explicit `serialVersionUID` .
- Enum types have a `serialVersionUID` of `0L` .
- Changing a class's structure without updating `serialVersionUID` can break compatibility.

### Annotated Complete Code Examples

**Example 1: Explicit serialVersionUID**

```java
import java.io.*;

class Employee implements Serializable {
    private static final long serialVersionUID = 1L;

    private String name;
    private double salary;

    // If you later add a field, keep serialVersionUID = 1L for compatibility
    // private String department; // New field

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    @Override
    public String toString() {
        return "Employee{name='" + name + "', salary=" + salary + "}";
    }
}
```

**Expected Output:** The class can be serialized and deserialized reliably across versions.

**Why This Output Occurs:** The explicit `serialVersionUID` ensures that the class is recognized as compatible even if the class definition changes slightly .

### Real-World Cases

- **Library evolution**: Adding fields while maintaining backward compatibility.
- **Distributed systems**: Ensuring all nodes have compatible class versions.
- **Database storage**: Persisting serialized objects with version tracking.

### References

- OpenJDK – Serializable (serialVersionUID) - https://cr.openjdk.org/~jboes/webrevs/8231186/webrev.02/src/java.base/share/classes/java/io/Serializable.java.sdiff.html
- Oracle – Class Descriptors (serialVersionUID) - https://docs.oracle.com/en/java/javase/11/docs/specs/serialization/class.html


## Core Concept 6: Modern Alternatives and Object Filters

### Definitions

**Core Definition**: Modern alternatives to native Java serialization use structured, cross-platform formats (JSON, XML, Protocol Buffers) that offer better security, performance, and interoperability. **Object filters** are security mechanisms that control which classes can be deserialized.

**Technical Definition**: The creator of Java, Joshua Bloch, recommends avoiding `Serializable` due to security risks and complexity . Alternatives include JSON (Jackson, Gson), Protocol Buffers (protobuf), XML, and Kryo . `ObjectInputFilter` (Java 9+) provides a way to allow or reject classes during deserialization .

**Beginner-Friendly Explanation**: Instead of using Java's built-in "save game" format (which is fragile and dangerous), you use a universal format like JSON (which any language can read) or Protocol Buffers (which is compact and fast).

### Purposes

- To improve security by avoiding deserialization of untrusted data.
- To enable cross-language data exchange.
- To improve performance and reduce payload size.
- To provide better versioning and schema evolution.

### Syntax Rules and Structure

**Comparison Table:**

| Method | Format | Pros | Cons | Use Case |
|--------|--------|------|------|----------|
| JDK Native | Binary | Built-in, RMI | Security risks, cross-language poor | RMI, JVM-internal |
| Jackson | JSON/XML | High performance, rich ecosystem | Requires configuration | REST APIs |
| Gson | JSON | Easy to use | Less customizable | Lightweight services |
| Protobuf | Binary | Small, fast, cross-language | Requires .proto definition | Microservices, RPC |
| Kryo | Binary | Very fast, easy | Not cross-language | High-performance caching  |

**ObjectInputFilter Syntax:**
```java
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "example.*;java.base/*;!*"
);
ObjectInputFilter.Config.setSerialFilter(filter);
```

**Syntax Rules:**
- Filters are configured via system properties, security properties, or programmatically .
- The filter pattern `!*` rejects everything not explicitly allowed.
- `ObjectInputFilter.Config.createFilter()` creates a pattern-based filter .

**Constraints and Limitations:**
- Filters are not enabled by default .
- Built-in filters for RMI should be extended, not relied upon .
- JSON libraries have their own security considerations (e.g., polymorphic deserialization).

### Annotated Complete Code Examples

**Example 1: Jackson JSON Serialization**

```java
import com.fasterxml.jackson.databind.ObjectMapper;

class UserDTO {
    public String name;
    public int age;
    public UserDTO() {} // Needed for Jackson
    public UserDTO(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

public class JacksonDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        UserDTO user = new UserDTO("Alice", 30);

        // Serialize to JSON
        String json = mapper.writeValueAsString(user);
        System.out.println("JSON: " + json);

        // Deserialize from JSON
        UserDTO restored = mapper.readValue(json, UserDTO.class);
        System.out.println("Restored: " + restored.name + ", " + restored.age);
    }
}
```

**Expected Output:**
```
JSON: {"name":"Alice","age":30}
Restored: Alice, 30
```

**Why This Output Occurs:** Jackson converts the object to a JSON string and back. No Java serialization is involved .

### Real-World Cases

- **REST APIs**: JSON is the standard for web services .
- **Microservices**: Protobuf for efficient inter-service communication .
- **High-performance caching**: Kryo or FST for JVM-internal serialization .
- **Security-critical systems**: Avoid native serialization entirely .

### References

- Yisu – Java API接口数据序列化方法有哪些 - https://www.yisu.com/jc/1067469.html
- Team IT Security – Serialização em Java na atualidade e novas abordagens em 2025 - https://tsecurity.de/de/2611686/IT+Programmierung/Serializa%C3%A7%C3%A3o+em+Java+na+atualidade+e+novas+abordagens+em+2025/de/2313993/Inhalte%20hinzuf%C3%BCgen/
- Oracle – ObjectInputFilter (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/ObjectInputFilter.html
- Oracle – Serialization Filtering - https://docs.oracle.com/javase/8/docs/technotes/guides/serialization/filters/serialization-filtering.html


## References

- SEI CERT – Serialization (Libraries) - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20241120-201124-1529-19/Libraries_7bb97f4cbd5847e58953970f6dd21802-201124-1529-20.pdf
- OpenJDK – State of Serialization - https://mail.openjdk.org/pipermail/core-libs-dev/2014-July/028000.html
- OpenJDK – Serialization Architecture - https://cr.openjdk.org/~ihse/JDK-8179556-add-legal-copyright-html/webrev.01/jdk/src/java.base/share/specs/serialization/serial-arch.md.sdiff.html
- Oracle – Addressing Serialization Vulnerabilities - https://docs.oracle.com/en/java/javase/21/core/addressing-serialization-vulnerabilities.html
- OpenJDK – Serializable (Java 9+) - https://cr.openjdk.org/~jboes/webrevs/8231186/webrev.02/src/java.base/share/classes/java/io/Serializable.java.sdiff.html
- Oracle – ObjectInputFilter (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/ObjectInputFilter.html
- Yisu – Java API接口数据序列化方法有哪些 - https://www.yisu.com/jc/1067469.html
- Apache – Serializable (Harmony) - https://svn.apache.org/repos/asf/harmony/enhanced/java/branches/java6/classlib/modules/luni/src/main/java/java/io/Serializable.java
- Princeton – Security in Object Serialization - https://www.cs.princeton.edu/courses/archive/fall97/cs461/jdkdocs/guide/serialization/spec/security.doc.html
- Oracle – Serialization Filtering - https://docs.oracle.com/javase/8/docs/technotes/guides/serialization/filters/serialization-filtering.html
- OpenJDK – Enum serialVersionUID - https://cr.openjdk.org/~darcy/8234381.0/jdk.patch
- Oracle – ObjectInputFilter (Java 17) - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/ObjectInputFilter.html
- Team IT Security – Serialização em Java na atualidade e novas abordagens em 2025 - https://tsecurity.de/de/2611686/IT+Programmierung/Serializa%C3%A7%C3%A3o+em+Java+na+atualidade+e+novas+abordagens+em+2025/de/2313993/Inhalte%20hinzuf%C3%BCgen/
- Caltech – Tag Interfaces and RMI - https://courses.cms.caltech.edu/cs11/material/advjava/lectures/cs11-advjava-lec6.pdf
- Google – Serializable (Android) - https://android.googlesource.com/platform/libcore.git/+/617f435b7decc07c3721b45edb0f4e7c4c2e54a5/luni/src/main/java/java/io/Serializable.java
- Max-Planck – Serialization Architecture (Transient) - https://resources.mpi-inf.mpg.de/d5/teaching/ss05/is05/java/jdk1.3/guide/serialization/spec/serial-arch.doc12.html
- Oracle – シリアライズ・フィルタリング - https://docs.oracle.com/javase/jp/8/docs/technotes/guides/serialization/filters/serialization-filtering.html
- Oracle – Class Descriptors (serialVersionUID) - https://docs.oracle.com/en/java/javase/11/docs/specs/serialization/class.html
- OpenJDK – ObjectInputFilter (Filter Factory) - https://cr.openjdk.org/~rriggs/filter-factory/java.base/java/io/ObjectInputFilter.html