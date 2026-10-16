# Java Properties: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `java.util.Properties` class represents a persistent set of configuration properties, where each key and its corresponding value is a string. Properties can be saved to a stream or loaded from a stream, making them a simple and portable way to externalise application configuration.

**Technical Definition:** `Properties` extends `Hashtable<Object,Object>` and is a specialised map-like structure designed for string-based key-value configuration. It provides methods for loading key/value pairs from a stream, retrieving a value from its key, listing keys and values, enumerating over keys, and saving properties to a stream. The class supports two storage formats: a simple line-oriented text format (`.properties`) and an XML format. The `load(Reader)`/`store(Writer, String)` methods use a character-based stream, while the `load(InputStream)`/`store(OutputStream, String)` methods use ISO 8859-1 encoding by default, with Unicode escapes for characters outside that range. Since Java 9, property files are encoded as UTF-8 by default, though the `load(InputStream)` method still assumes ISO-8859-1 unless a `Reader` is used.

**Beginner-Friendly Explanation:** A `Properties` object is like a dictionary or a settings panel in your application. You store configuration values as key-value pairs — for example, `database.url=jdbc:mysql://localhost:3306/mydb` or `app.name=MyApplication`. Instead of hard-coding these values in your Java source code, you put them in a separate `.properties` file that can be edited without recompiling. The `Properties` class reads that file and makes the values available to your program. This keeps your code clean and your configuration flexible across different environments (development, testing, production).

### Key Characteristics

- **String-Based:** Both keys and values are strings; type conversion (to int, boolean, etc.) is the application's responsibility.
- **Persistent:** Properties can be saved to and loaded from streams, making them suitable for storing configuration across application restarts.
- **Map-Like:** Extends `Hashtable`, providing familiar map operations like `get`, `put`, `remove`, and `containsKey`.
- **Default Support:** A `Properties` object can hold a reference to another `Properties` object as its "defaults", searched when a key is not found in the original.
- **Two Storage Formats:** Classic `.properties` text format and XML format (`loadFromXML`/`storeToXML`).
- **Encoding Evolution:** Historically ISO-8859-1; since Java 9, UTF-8 is the default for property files, with automatic fallback to ISO-8859-1 if needed.
- **Security-Sensitive:** Access to properties is subject to approval by the current security manager; storing secrets in plaintext is a critical security risk.
- **Internationalisation-Ready:** Works with `ResourceBundle` for locale-specific configuration and UI strings.

### Prerequisites

- Basic understanding of Java classes and objects
- Familiarity with Java I/O streams (`InputStream`, `OutputStream`, `Reader`, `Writer`)
- Knowledge of the `java.util` package
- Understanding of exceptions and try-with-resources

### Related Programming Areas

- **Configuration Management:** Externalising application settings from code.
- **Internationalisation (i18n):** Using `ResourceBundle` with `.properties` files for locale-specific strings.
- **Build Tools:** Maven and Gradle use `.properties` files for project metadata and configuration.
- **Spring Boot:** `application.properties` is the default configuration format for Spring Boot applications.
- **Security:** Managing secrets, encryption, and access control for configuration files.

### Core Concepts / Features

---

## 1. `Properties`

### Definitions

**Core Definition:** `Properties` is a Java class in the `java.util` package that represents a persistent set of string-based key-value configuration pairs.

**Technical Definition:** `Properties` extends `Hashtable<Object,Object>` and is a specialised map-like structure designed for string-based configuration. It provides methods for loading key/value pairs from a stream, retrieving values by key, listing keys and values, enumerating over keys, and saving properties to a stream. The class supports two storage formats: a simple line-oriented text format and an XML format. The `load(Reader)` and `store(Writer, String)` methods use character-based streams with configurable encoding. Since Java 9, property files are encoded as UTF-8 by default.

**Beginner-Friendly Explanation:** The `Properties` class is like a digital filing cabinet for your application's settings. You give it a key (like a label) and a value (the setting), and it stores them together. Later, you can look up the value by its key. The cabinet can be saved to a file and loaded back, so your settings persist even after the program closes.

### Purposes

- To externalise application configuration from source code
- To persist key-value settings across application restarts
- To provide a simple, string-based configuration mechanism without external dependencies
- To support layered configuration through defaults and overrides
- To enable internationalisation through integration with `ResourceBundle`
- To load configuration from the classpath, filesystem, or other streams

### Syntax Rules and Structure

**Complete General Syntax (Creating and Using Properties):**

```java
// Create a Properties object
Properties props = new Properties();

// Set properties
props.setProperty("db.url", "jdbc:mysql://localhost:3306/mydb");
props.setProperty("db.user", "admin");
props.setProperty("db.password", "secret");

// Get properties with defaults
String url = props.getProperty("db.url");
String timeout = props.getProperty("db.timeout", "30");  // default "30"

// Check existence
boolean hasUser = props.containsKey("db.user");

// Remove a property
props.remove("db.password");

// Iterate over property names
for (String name : props.stringPropertyNames()) {
    System.out.println(name + " = " + props.getProperty(name));
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `new Properties()` | Creates an empty property list |
| `setProperty(String, String)` | Adds or updates a property |
| `getProperty(String)` | Retrieves a value by key (returns `null` if absent) |
| `getProperty(String, String)` | Retrieves a value with a default fallback |
| `containsKey(Object)` | Checks if a key exists |
| `remove(Object)` | Removes a property |
| `stringPropertyNames()` | Returns a `Set<String>` of all keys |
| `size()` | Returns the number of properties |

**Syntax Rules:**

- Use `setProperty()` (not `put()`) to ensure keys and values are strings. Using `put()` with non-string keys or values is strongly discouraged and can cause `store()` to fail.
- Keys and values are case-sensitive.
- The default properties mechanism (`Properties defaults`) is searched when a key is not found in the main property list.

**Constraints and Limitations:**

- `Properties` does not support nested or hierarchical structures; keys use dots for namespacing by convention only.
- Type conversion is not automatic; values must be parsed manually.
- `Properties` is not thread-safe for concurrent modification; use external synchronisation if needed.
- Access to properties is subject to the security manager.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Properties Usage**

```java
// File: PropertiesBasicDemo.java
import java.util.Properties;

public class PropertiesBasicDemo {
    public static void main(String[] args) {
        // Step 1: Create a Properties object
        Properties config = new Properties();

        // Step 2: Set configuration values
        config.setProperty("app.name", "MyApplication");
        config.setProperty("app.version", "1.0.0");
        config.setProperty("server.port", "8080");
        config.setProperty("server.host", "localhost");

        // Step 3: Retrieve values
        String appName = config.getProperty("app.name");
        String port = config.getProperty("server.port");
        String timeout = config.getProperty("server.timeout", "30"); // default

        System.out.println("App: " + appName);        // Output: App: MyApplication
        System.out.println("Port: " + port);          // Output: Port: 8080
        System.out.println("Timeout: " + timeout);    // Output: Timeout: 30

        // Step 4: Iterate over all properties
        System.out.println("All properties:");
        for (String key : config.stringPropertyNames()) {
            System.out.println("  " + key + " = " + config.getProperty(key));
        }
    }
}
```

**Expected Output:**
```
App: MyApplication
Port: 8080
Timeout: 30
All properties:
  app.name = MyApplication
  app.version = 1.0.0
  server.port = 8080
  server.host = localhost
```

**Step-by-Step Explanation:**

1. `new Properties()` creates an empty configuration map.
2. `setProperty()` adds key-value pairs as strings.
3. `getProperty("app.name")` retrieves the value; `getProperty("server.timeout", "30")` returns the default `"30"` because the key is absent.
4. `stringPropertyNames()` returns all keys, which are iterated to print each property.

### Real-World Cases

- **Database Configuration:** Storing JDBC URLs, usernames, and connection pool settings.
- **Application Settings:** Feature flags, timeouts, and service endpoints.
- **Build Metadata:** Version numbers, build timestamps, and environment identifiers.
- **Localisation:** Storing UI strings for different locales via `ResourceBundle`.

---

## 2. Properties File

### Definitions

**Core Definition:** A properties file is a plain text file with a `.properties` extension that contains key-value pairs in a simple line-oriented format.

**Technical Definition:** A properties file is a plain text configuration format built on `key=value` pairs. Each line sets a key to a value using one of the supported separators: equals sign (`=`), colon (`:`), or whitespace. Comments start with `#` or `!`, and blank lines are ignored. Leading and trailing whitespace around keys and values is generally trimmed, though escaping can preserve it. Keys can contain dots (`.`) to imply namespaces and hierarchy, and values can include spaces without quoting.

**Beginner-Friendly Explanation:** A properties file is like a settings file for your application. Each line has a setting name and its value, separated by `=` or `:`. You can add comments with `#` to explain what each setting does. It's simple enough to edit by hand and works with any text editor.

### Purposes

- To externalise configuration from code for flexibility across environments
- To provide a human-readable and hand-editable configuration format
- To support environment-specific overrides (development, testing, production)
- To serve as the default configuration format for many Java frameworks
- To enable configuration changes without recompiling the application

### Syntax Rules and Structure

**Properties File Format:**

```properties
# Database configuration
jdbc.url = jdbc:postgresql://db.example.com:5432/app
jdbc.user = app_user
jdbc.password : s3cr3t
app.mode = production

# Server settings
server.port=8080
server.host=localhost

# Multi-line value (continuation with backslash)
app.description = This is a long description \
    that spans multiple lines.

# Escaped characters
path = C:\\Program Files\\MyApp
unicode.key = \u00E9\u00E8\u00EA
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `key=value` | Standard key-value pair with `=` separator |
| `key:value` | Alternative separator (`:`) |
| `key value` | Whitespace separator (legacy) |
| `# comment` or `! comment` | Comment line |
| `\` at end of line | Line continuation |
| `\uXXXX` | Unicode escape for non-Latin-1 characters |
| `\\`, `\=`, `\:`, `\#`, `\!` | Escaped special characters |

**Syntax Rules:**

- Keys and values are separated by `=`, `:`, or whitespace. Standardise on one style within a file.
- Comments begin with `#` or `!`.
- Leading and trailing whitespace around keys and values is trimmed unless escaped.
- Backslash (`\`) at the end of a line continues the value on the next line.
- Special characters (`=`, `:`, `#`, `!`, `\`) in keys or values must be escaped with a backslash.
- Non-Latin-1 characters can be written as `\uXXXX` Unicode escapes (for ISO-8859-1 encoding) or directly in UTF-8 since Java 9.

**Constraints and Limitations:**

- The format is flat; nesting requires dotted key names by convention.
- Values are always strings; type conversion is manual.
- Historically ISO-8859-1 encoding; Java 9+ defaults to UTF-8.
- No support for arrays, lists, or complex data types.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Reading a Properties File**

```java
// File: ReadPropertiesDemo.java
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class ReadPropertiesDemo {
    public static void main(String[] args) {
        Properties props = new Properties();

        // Load from classpath
        try (InputStream input = ReadPropertiesDemo.class
                .getClassLoader()
                .getResourceAsStream("config.properties")) {

            if (input == null) {
                System.out.println("Unable to find config.properties");
                return;
            }

            props.load(input);

            System.out.println("App: " + props.getProperty("app.name"));
            System.out.println("Port: " + props.getProperty("server.port"));
            System.out.println("Mode: " + props.getProperty("app.mode"));

        } catch (IOException e) {
            System.out.println("Error loading properties: " + e.getMessage());
        }
    }
}
```

**Expected Output (for `config.properties` with `app.name=MyApp`, `server.port=8080`, `app.mode=dev`):**
```
App: MyApp
Port: 8080
Mode: dev
```

**Step-by-Step Explanation:**

1. `new Properties()` creates an empty properties object.
2. `getResourceAsStream("config.properties")` loads the file from the classpath.
3. `props.load(input)` reads the key-value pairs from the stream.
4. `getProperty()` retrieves individual values by key.
5. The try-with-resources statement ensures the input stream is closed.

### Real-World Cases

- **Spring Boot:** `application.properties` for application configuration.
- **Maven:** `pom.properties` for build metadata.
- **Internationalisation:** `messages.properties` for UI strings.
- **Database Drivers:** Connection pool configuration files.

---

## 3. `.properties` Format

### Definitions

**Core Definition:** The `.properties` format is the classic line-oriented text format used by Java `Properties` files, defined by the `java.util.Properties` class specification.

**Technical Definition:** The classic `.properties` format uses a simple line-based syntax with `key=value` or `key:value` pairs and supports escaping for special characters and line continuations. This format historically uses ISO-8859-1 character encoding with Unicode escapes for characters outside that range. Since Java 9, the default encoding for property files is UTF-8, with automatic fallback to ISO-8859-1 if needed.

**Beginner-Friendly Explanation:** The `.properties` format is a simple way to store settings. Each line is a setting. You can use `=` or `:` to separate the setting name from its value. If you need to write a really long value, you can use a backslash at the end of the line to continue on the next line. Special characters like `=` or `#` need a backslash before them if they're part of a key or value.

### Purposes

- To provide a simple, human-readable format for key-value configuration
- To enable hand-editing of configuration without specialised tools
- To support comments for documentation
- To allow multi-line values through line continuation
- To support Unicode characters through escapes or UTF-8 encoding

### Syntax Rules and Structure

**Escaping Rules:**

| Character | Escape Sequence | Context |
|-----------|-----------------|---------|
| `\` | `\\` | Backslash in keys or values |
| `=` | `\=` | Equals sign in keys |
| `:` | `\:` | Colon in keys |
| `#` | `\#` | Hash in keys |
| `!` | `\!` | Exclamation in keys |
| `\t` | `\t` | Tab character |
| `\n` | `\n` | Newline character |
| `\r` | `\r` | Carriage return |
| `\f` | `\f` | Form feed |
| `\uXXXX` | `\uXXXX` | Unicode character |

**Syntax Rules:**

- Keys can contain any character except unescaped `=`, `:`, or whitespace (which act as separators).
- Values can contain any character except unescaped newlines (which end the value).
- Leading whitespace in values is trimmed unless escaped with `\`.
- Trailing whitespace in values is always trimmed unless escaped.
- Comments cannot appear after a key-value pair on the same line.

**Constraints and Limitations:**

- No support for nested structures; use dotted keys.
- No native support for numeric, boolean, or date types.
- Encoding complexities with ISO-8859-1 vs UTF-8.
- Multi-line values can be error-prone to edit.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Advanced Properties File Features**

```properties
# advanced.properties
# Database settings
jdbc.url=jdbc:mysql://localhost:3306/mydb
jdbc.user=admin
jdbc.password=secret

# Multi-line value using backslash continuation
app.description=This application manages \
    inventory and orders \
    for warehouse operations.

# Escaped characters
path=C:\\Program Files\\MyApp
special.key=value\=with\=equals

# Unicode escape (ISO-8859-1 compatible)
greeting=\u00C9\u00E8\u00EA

# Whitespace in keys and values
\ key\ with\ spaces\ =value with spaces
```

```java
// File: AdvancedPropertiesDemo.java
import java.io.InputStream;
import java.util.Properties;

public class AdvancedPropertiesDemo {
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();

        try (InputStream input = AdvancedPropertiesDemo.class
                .getClassLoader().getResourceAsStream("advanced.properties")) {
            props.load(input);
        }

        System.out.println("URL: " + props.getProperty("jdbc.url"));
        System.out.println("Description: " + props.getProperty("app.description"));
        System.out.println("Path: " + props.getProperty("path"));
        System.out.println("Special: " + props.getProperty("special.key"));
        System.out.println("Greeting: " + props.getProperty("greeting"));
        System.out.println("Spaced key: " + props.getProperty(" key with spaces "));
    }
}
```

**Expected Output:**
```
URL: jdbc:mysql://localhost:3306/mydb
Description: This application manages inventory and orders for warehouse operations.
Path: C:\Program Files\MyApp
Special: value=with=equals
Greeting: Éèê
Spaced key: value with spaces
```

**Step-by-Step Explanation:**

1. The multi-line value uses `\` at the end of each line to continue the value.
2. `\\` in the path produces a single backslash in the value.
3. `\=` escapes the equals sign so it is part of the value, not a separator.
4. `\u00C9` etc. produce the characters É, è, ê.
5. Escaped spaces (`\ `) preserve leading and trailing spaces in keys.

### Real-World Cases

- **Database Configuration:** Connection URLs and credentials.
- **Application Settings:** Feature flags and service endpoints.
- **Internationalisation:** Unicode strings for different locales.
- **Build Metadata:** Version and environment information.

---

## 4. Loading Properties

### Definitions

**Core Definition:** Loading properties is the process of reading key-value pairs from a file or stream into a `Properties` object.

**Technical Definition:** The `Properties` class provides multiple `load()` methods: `load(InputStream)` reads from a byte stream using ISO-8859-1 encoding; `load(Reader)` reads from a character stream with configurable encoding; `loadFromXML(InputStream)` reads from an XML format. The `load(Reader)` and `load(InputStream)` methods parse the simple line-oriented format specified in the class documentation.

**Beginner-Friendly Explanation:** Loading properties means reading your settings file and putting all the settings into a `Properties` object so your program can access them. You can load from a file on disk, from the classpath, or even from a string. Once loaded, you can ask for any setting by its key.

### Purposes

- To read configuration from external files at application startup
- To load properties from the classpath for portable configuration
- To merge multiple property sources (defaults + overrides)
- To parse properties from strings or other streams
- To load XML-formatted properties when needed

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Load from InputStream (ISO-8859-1)
Properties props = new Properties();
try (InputStream input = new FileInputStream("config.properties")) {
    props.load(input);
}

// Load from Reader (UTF-8)
try (Reader reader = Files.newBufferedReader(
        Paths.get("config.properties"), StandardCharsets.UTF_8)) {
    props.load(reader);
}

// Load from classpath
try (InputStream input = MyClass.class
        .getClassLoader().getResourceAsStream("config.properties")) {
    props.load(input);
}

// Load from XML
try (InputStream input = new FileInputStream("config.xml")) {
    props.loadFromXML(input);
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `load(InputStream)` | Reads from a byte stream using ISO-8859-1 encoding |
| `load(Reader)` | Reads from a character stream with specified encoding |
| `loadFromXML(InputStream)` | Reads from an XML properties document |
| `getResourceAsStream()` | Loads from the classpath |

**Syntax Rules:**

- Always use try-with-resources to ensure streams are closed.
- Use `load(Reader)` with a `BufferedReader` for UTF-8 encoding.
- Prefer loading from the classpath for portable, packaged applications.
- The `load()` methods clear any existing properties before loading.

**Constraints and Limitations:**

- `load(InputStream)` assumes ISO-8859-1; use `load(Reader)` for UTF-8.
- Loading from the classpath returns `null` if the resource is not found.
- Large property files consume memory proportionally.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Loading from Multiple Sources with Defaults**

```java
// File: LoadPropertiesDemo.java
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.Properties;
import java.io.Reader;

public class LoadPropertiesDemo {
    public static void main(String[] args) throws Exception {
        Properties defaults = new Properties();
        defaults.setProperty("server.port", "8080");
        defaults.setProperty("server.host", "localhost");
        defaults.setProperty("app.mode", "development");

        // Create a Properties with defaults
        Properties config = new Properties(defaults);

        // Load overrides from a file (UTF-8)
        Path path = Path.of("overrides.properties");
        Files.writeString(path, "server.port=9090\napp.mode=production\n",
            StandardCharsets.UTF_8);

        try (Reader reader = Files.newBufferedReader(path, StandardCharsets.UTF_8)) {
            config.load(reader);
        }

        // Keys not overridden fall back to defaults
        System.out.println("Port: " + config.getProperty("server.port")); // 9090
        System.out.println("Host: " + config.getProperty("server.host")); // localhost
        System.out.println("Mode: " + config.getProperty("app.mode"));    // production

        Files.deleteIfExists(path);
    }
}
```

**Expected Output:**
```
Port: 9090
Host: localhost
Mode: production
```

**Step-by-Step Explanation:**

1. A `Properties` object is created with default values.
2. A second `Properties` object is created with the defaults as its default source.
3. Overrides are loaded from a UTF-8 file using `load(Reader)`.
4. `getProperty("server.port")` returns `9090` (overridden); `getProperty("server.host")` returns `localhost` (from defaults).

### Real-World Cases

- **Spring Boot:** Loading `application.properties` from the classpath.
- **Configuration Layering:** Defaults from a base file, overrides from environment-specific files.
- **Testing:** Loading test-specific configuration from test resources.
- **Web Applications:** Loading configuration from `WEB-INF/classes`.

---

## 5. Saving Properties

### Definitions

**Core Definition:** Saving properties is the process of writing the key-value pairs from a `Properties` object to a file or stream.

**Technical Definition:** The `Properties` class provides `store(OutputStream, String)` and `store(Writer, String)` methods for writing properties in the classic `.properties` format, and `storeToXML(OutputStream, String, String)` for writing in XML format. The `store(Writer, String)` method writes characters through the specified encoding, allowing UTF-8 output without escapes.

**Beginner-Friendly Explanation:** Saving properties means writing your settings from a `Properties` object back to a file. You can save them in the classic `.properties` format or in XML. If you use a `Writer` with UTF-8 encoding, your non-Latin characters are saved directly without needing `\uXXXX` escapes.

### Purposes

- To persist configuration changes made at runtime
- To generate configuration files programmatically
- To export properties for backup or migration
- To create default configuration files on first run
- To save application state between sessions

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Store to OutputStream (ISO-8859-1 with escapes)
try (OutputStream output = new FileOutputStream("config.properties")) {
    props.store(output, "Application Configuration");
}

// Store to Writer (UTF-8, no escapes)
try (Writer writer = Files.newBufferedWriter(
        Path.of("config.properties"), StandardCharsets.UTF_8)) {
    props.store(writer, "Application Configuration");
}

// Store to XML
try (OutputStream output = new FileOutputStream("config.xml")) {
    props.storeToXML(output, "Application Configuration", StandardCharsets.UTF_8);
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `store(OutputStream, String)` | Writes ISO-8859-1 with Unicode escapes |
| `store(Writer, String)` | Writes with specified encoding (e.g., UTF-8) |
| `storeToXML(OutputStream, String, Charset)` | Writes XML format with encoding |

**Syntax Rules:**

- The comment string is written as a header comment in the output.
- `store()` writes properties in an unspecified order (hash table order).
- Use `store(Writer)` with UTF-8 for human-readable output of non-Latin characters.
- Always use try-with-resources to ensure streams are flushed and closed.

**Constraints and Limitations:**

- Property order is not preserved; if order matters, use a different format.
- `store()` includes the current timestamp in the comment header.
- XML format is more verbose but handles encoding more predictably.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Saving Properties to a UTF-8 File**

```java
// File: SavePropertiesDemo.java
import java.io.Writer;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.Properties;

public class SavePropertiesDemo {
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();
        props.setProperty("app.name", "Café Manager");
        props.setProperty("app.version", "2.0.0");
        props.setProperty("greeting", "Bonjour, ça va?");

        Path path = Path.of("output.properties");

        // Save with UTF-8 encoding (no Unicode escapes)
        try (Writer writer = Files.newBufferedWriter(path, StandardCharsets.UTF_8)) {
            props.store(writer, "Application Configuration — UTF-8");
        }

        // Read back to verify
        Properties loaded = new Properties();
        try (var reader = Files.newBufferedReader(path, StandardCharsets.UTF_8)) {
            loaded.load(reader);
        }

        System.out.println("Name: " + loaded.getProperty("app.name"));
        System.out.println("Greeting: " + loaded.getProperty("greeting"));

        Files.deleteIfExists(path);
    }
}
```

**Expected Output:**
```
Name: Café Manager
Greeting: Bonjour, ça va?
```

**Step-by-Step Explanation:**

1. Properties are set with accented characters.
2. `store(Writer)` writes with UTF-8 encoding, preserving the characters directly.
3. The file is read back with `load(Reader)` using UTF-8.
4. The accented characters are correctly preserved.

### Real-World Cases

- **Configuration Generators:** Tools that generate default configuration files.
- **Runtime Configuration:** Saving user preferences or application state.
- **Migration Scripts:** Exporting properties for migration to a new system.
- **Testing:** Creating temporary configuration files for tests.

---

## 6. Properties Best Practices

### Definitions

**Core Definition:** Properties best practices are guidelines for using `java.util.Properties` effectively, safely, and maintainably in Java applications.

**Technical Definition:** Best practices derive from authoritative sources including Oracle's Java Tutorials and community experience. Key principles include using `setProperty()` instead of `put()`, choosing `Reader`/`Writer`-based methods for encoding control, layering configuration with defaults and overrides, and never storing secrets in plaintext.

**Beginner-Friendly Explanation:** Best practices help you avoid common mistakes when working with properties files. They're like a checklist for doing things the right way — using the correct methods, handling encoding properly, keeping secrets safe, and organising your configuration sensibly.

### Purposes

- To ensure correct and predictable encoding behaviour
- To prevent security vulnerabilities from plaintext secrets
- To maintain clean, maintainable configuration files
- To avoid common pitfalls like silent failures and encoding corruption
- To support modern deployment patterns like environment-based overrides

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| Use `setProperty()` not `put()` | Ensures keys and values are strings; `put()` with non-strings can break `store()`. |
| Prefer `load(Reader)`/`store(Writer)` | Gives explicit control over encoding (UTF-8). |
| Never store secrets in plaintext | Use environment variables, Vault, or encryption instead. |
| Use dotted keys for namespacing | e.g., `db.url`, `db.user` for logical grouping. |
| Layer configuration | Defaults file → environment-specific file → environment variables → system properties. |
| Validate configuration at startup | Fail fast if required properties are missing or invalid. |
| Use `getProperty(key, default)` | Provide sensible defaults instead of returning `null`. |
| Mask secrets when logging | Never log passwords or tokens from properties. |
| Keep environment configs separate | Use `application-dev.properties`, `application-prod.properties`, etc. |
| Document your properties | Add comments explaining each setting's purpose and valid values. |
| Avoid properties for complex data | Use YAML or JSON for nested or structured configuration. |
| Test with different encodings | Ensure your build and deployment handle UTF-8 consistently. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: PropertiesBestPracticesDemo.java
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.Properties;

public class PropertiesBestPracticesDemo {
    public static void main(String[] args) throws Exception {
        // BEST PRACTICE: Use setProperty() for string safety
        Properties config = new Properties();
        config.setProperty("app.name", "BestPracticesApp");
        config.setProperty("db.url", "jdbc:postgresql://localhost:5432/app");
        config.setProperty("db.user", "app_user");
        // BEST PRACTICE: Do NOT store passwords in plaintext
        // config.setProperty("db.password", "secret"); // ❌ NEVER DO THIS

        // BEST PRACTICE: Provide defaults with getProperty
        String timeout = config.getProperty("db.timeout", "30");
        String maxConn = config.getProperty("db.maxConnections", "10");

        System.out.println("Timeout: " + timeout);      // Output: Timeout: 30
        System.out.println("Max Connections: " + maxConn); // Output: Max Connections: 10

        // BEST PRACTICE: Save with UTF-8 using Writer
        Path path = Path.of("best_practices.properties");
        try (Writer writer = Files.newBufferedWriter(path, StandardCharsets.UTF_8)) {
            config.store(writer, "Best Practices Configuration");
        }

        // BEST PRACTICE: Load with UTF-8 using Reader
        Properties loaded = new Properties();
        try (Reader reader = Files.newBufferedReader(path, StandardCharsets.UTF_8)) {
            loaded.load(reader);
        }

        // BEST PRACTICE: Validate required properties
        String required = loaded.getProperty("app.name");
        if (required == null || required.isEmpty()) {
            throw new IllegalStateException("Missing required property: app.name");
        }
        System.out.println("Validated: " + required);  // Output: Validated: BestPracticesApp

        Files.deleteIfExists(path);
    }
}
```

**Expected Output:**
```
Timeout: 30
Max Connections: 10
Validated: BestPracticesApp
```

**Step-by-Step Explanation:**

1. `setProperty()` is used consistently instead of `put()`.
2. Passwords are deliberately not stored in the properties file (a comment warns against it).
3. `getProperty(key, default)` provides fallback values.
4. `store(Writer)` with UTF-8 and `load(Reader)` with UTF-8 ensure encoding consistency.
5. Required properties are validated after loading.

### Real-World Cases

- **Spring Boot Applications:** Layered `application.properties` with profile-specific overrides.
- **Microservices:** Externalised configuration with environment variable overrides.
- **Enterprise Applications:** Encrypted properties for database credentials using Jasypt.
- **Internationalisation:** `ResourceBundle` with locale-specific `.properties` files.

---

## References Links

- Class Properties — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/Properties.html
- Properties — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/environment/properties.html
- Properties File Format: Syntax, Encoding, Best Practices — Cleverence - https://www.cleverence.com/articles/oracle-documentation/properties-file-format-7482/
- Java Properties: The Essential Guide to java.util.Properties — Cleverence - https://www.cleverence.com/articles/oracle-documentation/properties-essential-java-classes-7283/
- Mastering java.util.Properties in Java 21 — Cleverence - https://www.cleverence.com/articles/oracle-documentation/properties-java-se-21-jdk-21-4182/
- Release Note: UTF-8 based Properties Files — OpenJDK - https://bugs.openjdk.org/browse/JDK-8175225
- PropertyResourceBundle — Java Platform SE API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/PropertyResourceBundle.html
- ResourceBundle — Android Developers - https://developer.android.com/reference/java/util/ResourceBundle
- Spring YAML vs Properties — Baeldung - https://www.baeldung.com/spring-yaml-vs-properties
- CWE-260: Password in Configuration File — Plexicus - https://www.plexicus.ai