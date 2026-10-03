# JSON Processing & Modern Object Mapping: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition
JSON Processing & Modern Object Mapping is the practice of converting Java objects to JSON (serialization) and JSON back to Java objects (deserialization) using libraries like Jackson, with modern Java Records serving as immutable, boilerplate-free data carriers.

### Technical Definition
JSON (JavaScript Object Notation) is a lightweight, language-independent data interchange format. Jackson is a high-performance, Free/Open Source JSON processing library that provides the `ObjectMapper` class as the central API for serialization and deserialization. Jackson 2.12+ natively supports Java Records, but requires the `-parameters` compiler flag to retain parameter names for deserialization. The library also provides a tree model (`JsonNode`) for dynamic JSON navigation and a streaming API (`JsonParser`) for memory-efficient processing of large payloads.

### Beginner-Friendly Explanation
JSON is the "universal language" of web APIs and configuration files. When your Java application talks to a REST API, it sends and receives JSON. Jackson is the translator that converts your Java objects into JSON strings and back. Java Records make this translation cleaner because they are immutable data classes with less boilerplate code. Think of Jackson as a bilingual assistant: you give it a Java object, it writes the JSON; you give it JSON, it builds the Java object.

### Key Characteristics
- **Declarative**: Annotations control serialization/deserialization behavior
- **Type-Aware**: Maps JSON types to Java types automatically
- **Configurable**: Naming strategies, date formats, and null handling are customizable
- **Efficient**: Streaming API processes large JSON without loading entire trees
- **Record-Supporting**: Native support for Java Records since Jackson 2.12
- **Thread-Safe**: `ObjectMapper` is thread-safe when not modified after configuration

### Prerequisites
- Basic Java programming (classes, interfaces, generics)
- Familiarity with Java Collections (List, Map, Set)
- Understanding of Java Records (Java 16+)
- Maven or Gradle for dependency management
- Basic knowledge of annotations

### Related Programming Areas
- **REST APIs**: Spring Boot, JAX-RS for JSON request/response handling
- **Configuration Management**: YAML, properties files parsed via Jackson
- **Data Serialization**: Persistence, caching, message queues
- **Testing**: JSON assertions with JsonPath or AssertJ

### Core Concepts Overview
1. **JSON Anatomy**: Mapping JSON types (objects, arrays, strings, numbers, booleans, null) to Java types
2. **Jackson Framework**: Configuring the industry-standard `ObjectMapper`
3. **Serialization & Deserialization**: Converting Java objects to JSON strings and parsing payloads back
4. **Modern DTOs with Records**: Leveraging Java Records as immutable, boilerplate-free data carriers
5. **Customizing Mapping**: Naming strategies, ignoring unknown properties, and date/time formatting
6. **Tree Model & Streaming**: Parsing dynamic JSON with `JsonNode` and optimizing large streams with `JsonParser`

---

## Core Concept 1: JSON Anatomy

### Definitions
**Core Definition**: JSON anatomy refers to the structural mapping between JSON's six data types (objects, arrays, strings, numbers, booleans, null) and their corresponding Java types.

**Technical Definition**: JSON defines six value types: **object** (key-value pairs), **array** (ordered list), **string** (text), **number** (integer or floating-point), **boolean** (true/false), and **null**. When Jackson deserializes JSON into Java objects, it maps these types according to a natural mapping: JSON objects become `Map<String, Object>` or POJOs; JSON arrays become `List<Object>` or `Object[]`; JSON numbers become `Integer`, `Long`, or `Double`; JSON strings become `String`; JSON booleans become `Boolean`; and JSON null becomes `null`.

**Beginner-Friendly Explanation**: JSON is like a universal shipping container format. Objects are boxes with labeled compartments. Arrays are lists of items. Strings are text labels. Numbers are numeric values. Booleans are on/off switches. Null is an empty compartment. Jackson knows how to unpack each type into the right Java container.

### Purposes
- To understand the fundamental type system shared between JSON and Java
- To predict how JSON payloads will map to Java objects
- To design DTOs that align with JSON structures
- To diagnose type mismatch errors during deserialization
- To choose the right Java type for each JSON field

### Syntax Rules and Structure

#### Complete General Syntax: JSON to Java Type Mapping
```
JSON TYPE → JAVA TYPE MAPPING
│
├── JSON Object { }
│   ├── → Map<String, Object>
│   ├── → POJO / Record (typed fields)
│   └── → JsonNode (tree model)
│
├── JSON Array [ ]
│   ├── → List<Object>
│   ├── → Object[]
│   └── → JsonNode (tree model)
│
├── JSON String " "
│   ├── → String
│   ├── → char[] (with custom deserializer)
│   └── → Enum (with @JsonValue)
│
├── JSON Number
│   ├── Integer range → Integer / int
│   ├── Long range → Long / long
│   ├── Decimal → Double / double
│   └── BigDecimal → BigDecimal (with config)
│
├── JSON Boolean true/false
│   └── → Boolean / boolean
│
└── JSON null
    └── → null (for object types)
```

#### Component Breakdown
| JSON Type | Java Type (Default) | Java Type (Typed) |
|-----------|---------------------|-------------------|
| Object | `LinkedHashMap<String, Object>` | POJO / Record |
| Array | `ArrayList<Object>` | `List<T>` / `T[]` |
| String | `String` | `String` / `Enum` |
| Number (int) | `Integer` | `int` / `Integer` |
| Number (decimal) | `Double` | `double` / `BigDecimal` |
| Boolean | `Boolean` | `boolean` / `Boolean` |
| null | `null` | `null` / `Optional<T>` |

#### Syntax Rules
- Use typed DTOs (Records/POJOs) for structured JSON
- Use `JsonNode` for dynamic or unknown JSON structures
- Use `List<T>` for homogeneous arrays; `Object[]` for mixed arrays
- Configure `DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS` for precision
- Use `Optional<T>` for fields that may be absent or null

#### Constraints and Limitations
- Java generics are erased at runtime; use `TypeReference` for nested generics
- `int` cannot represent JSON null; use `Integer` for nullable numbers
- `BigDecimal` requires explicit configuration or `@JsonFormat(shape = JsonFormat.Shape.STRING)`
- Enum mapping requires `@JsonValue` or `@JsonCreator`

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: JSON to Java Type Mapping (Using Maven (Recommended))
Maven is the easiest way to handle the Jackson dependency because it downloads the required files for you automatically.

**Step 1: Install a Java Development Kit (JDK)**
Ensure you have JDK 17 or higher installed, as the code uses Java Records and text blocks ("""). Check your version by running this in your terminal/command prompt:
```bash
java -version
```

**Step 2: Create a project folder structure**
Create a new folder for your project and set up the standard Maven folder layout:
```
JsonDemo/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── JsonTypeMappingDemo.java
```

**Step 3: Create the pom.xml file**
In the root JsonDemo folder, create a file named pom.xml and paste the following configuration. This tells Maven to download Jackson:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://apache.org"
         xmlns:xsi="http://w3.org"
         xsi:schemaLocation="http://apache.org http://apache.org">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>json-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Jackson Databind (includes core and annotations) -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.15.2</version>
        </dependency>
    </dependencies>
</project>
```

**Step 4: Create the Java file**
Navigate to src/main/java/ and create a file named JsonTypeMappingDemo.java. Paste your exact code into it:
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.core.type.TypeReference;
import java.util.*;

public class JsonTypeMappingDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        // JSON with all six types
        String json = """
        {
          "name": "Alice",
          "age": 30,
          "salary": 95000.50,
          "active": true,
          "nickname": null,
          "skills": ["Java", "SQL", "JSON"]
        }
        """;

        // Map to typed Record
        record Person(
            String name, int age, double salary, 
            boolean active, String nickname, List<String> skills) {}
        Person person = mapper.readValue(json, Person.class);

        System.out.println("Name     : " + person.name());
        System.out.println("Age      : " + person.age());
        System.out.println("Salary   : " + person.salary());
        System.out.println("Active   : " + person.active());
        System.out.println("Nickname : " + person.nickname());
        System.out.println("Skills   : " + person.skills());

        // Map to generic Map (untyped)
        Map<String, Object> map = mapper.readValue(json, new TypeReference<Map<String, Object>>() {});
        System.out.println("\nUntyped map:");
        map.forEach((k, v) -> {
            String className = (v == null) ? "null" : v.getClass().getSimpleName();
            System.out.println("  " + k + " -> " + v + " (" + className + ")");
        });
    }
}
```
(Note: I added a small check (v == null) ? "null" : ... to prevent a NullPointerException when the program prints the class name for the null nickname).

**Step 5: Compile and Run**
Open your terminal inside the root JsonDemo folder (where pom.xml lives) and run:
```bash
mvn compile exec:java -Dexec.mainClass="JsonTypeMappingDemo"
```

**Expected Output**:
```
Name     : Alice
Age      : 30
Salary   : 95000.5
Active   : true
Nickname : null
Skills   : [Java, SQL, JSON]

Untyped map:
  name -> Alice (String)
  age -> 30 (Integer)
  salary -> 95000.5 (Double)
  active -> true (Boolean)
  nickname -> null
  skills -> [Java, SQL, JSON] (ArrayList)
```
**Why This Output**: Jackson maps JSON types to Java types automatically. The typed Record forces specific types (`int`, `double`, `boolean`). The untyped `Map` reveals Jackson's default mapping: numbers become `Integer` or `Double`, strings become `String`, booleans become `Boolean`.

### Real-World Cases
- **REST APIs**: Spring Boot controllers deserialize JSON request bodies into DTOs
- **Configuration**: JSON config files mapped to Java Records
- **Message Queues**: Kafka/RabbitMQ messages serialized as JSON
- **Data Pipelines**: ETL tools parse JSON into typed Java objects

### References
- Jackson Databind Documentation - https://github.com/FasterXML/jackson-databind
- Baeldung: Intro to the Jackson ObjectMapper - https://www.baeldung.com/jackson-object-mapper-tutorial

---

## Core Concept 2: Jackson Framework

### Definitions
**Core Definition**: The Jackson framework is a high-performance JSON processing library centered on the `ObjectMapper` class, which provides thread-safe serialization and deserialization between Java objects and JSON.

**Technical Definition**: Jackson consists of three core modules: `jackson-core` (streaming API), `jackson-annotations` (annotations), and `jackson-databind` (`ObjectMapper` and data binding). The `ObjectMapper` class is thread-safe once configured and should be reused as a singleton. It provides `writeValue()`/`writeValueAsString()` for serialization and `readValue()`/`readTree()` for deserialization.

**Beginner-Friendly Explanation**: Jackson is the industry-standard JSON library for Java. Think of `ObjectMapper` as a universal translator that sits in your application. You create it once, configure it for your needs, and then use it everywhere. It's fast, reliable, and handles all the tedious type conversion automatically.

### Purposes
- To provide a single, reusable entry point for all JSON operations
- To centralize JSON configuration (naming, dates, null handling)
- To ensure thread-safe concurrent JSON processing
- To integrate with Spring Boot's auto-configuration
- To support both tree model and streaming APIs

### Syntax Rules and Structure

#### Complete General Syntax: ObjectMapper Configuration
```
OBJECTMAPPER CONFIGURATION
│
├── 1. Create (once, reuse)
│   └── ObjectMapper mapper = new ObjectMapper();
│
├── 2. Configure Features
│   ├── mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
│   ├── mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)
│   └── mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)
│
├── 3. Register Modules
│   ├── mapper.registerModule(new JavaTimeModule())
│   └── mapper.registerModule(new ParameterNamesModule())
│
├── 4. Serialize
│   ├── String json = mapper.writeValueAsString(obj)
│   └── mapper.writeValue(new File("out.json"), obj)
│
└── 5. Deserialize
    ├── T obj = mapper.readValue(json, Class<T>)
    └── JsonNode node = mapper.readTree(json)
```

#### Component Breakdown
| Component | Maven Artifact | Purpose |
|-----------|---------------|---------|
| `jackson-core` | `com.fasterxml.jackson.core:jackson-core` | Streaming API |
| `jackson-annotations` | `com.fasterxml.jackson.core:jackson-annotations` | Annotations |
| `jackson-databind` | `com.fasterxml.jackson.core:jackson-databind` | `ObjectMapper` |
| `jackson-datatype-jsr310` | `com.fasterxml.jackson.datatype:jackson-datatype-jsr310` | Java 8 Date/Time |
| `jackson-module-parameter-names` | `com.fasterxml.jackson.module:jackson-module-parameter-names` | Record support |

#### Syntax Rules
- Add `jackson-databind` dependency; it transitively includes `jackson-core` and `jackson-annotations`
- Create `ObjectMapper` once and reuse (thread-safe)
- Register `JavaTimeModule` for `java.time` types
- Register `ParameterNamesModule` for Records
- Never modify `ObjectMapper` after it's shared across threads

#### Constraints and Limitations
- `ObjectMapper` is thread-safe only if not modified after publication
- Jackson 3.0 (in development) changes default date handling
- Records require the `-parameters` compiler flag
- No built-in support for Kotlin data classes (requires `jackson-module-kotlin`)

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Configured ObjectMapper
```java
// JacksonConfigDemo.java
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.module.paramnames.ParameterNamesModule;
import java.time.LocalDateTime;

public class JacksonConfigDemo {
    
    record Event(String eventName, LocalDateTime timestamp) {}
    
    public static void main(String[] args) throws Exception {
        // Configure ObjectMapper once
        ObjectMapper mapper = new ObjectMapper();
        
        // Register modules for Records and Java 8 Date/Time
        mapper.registerModule(new ParameterNamesModule());
        mapper.registerModule(new JavaTimeModule());
        
        // Disable timestamp-based dates
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        
        // Ignore unknown properties
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        
        // Serialize a Record
        Event event = new Event("Deployment", LocalDateTime.of(2026, 10, 3, 14, 30));
        String json = mapper.writeValueAsString(event);
        System.out.println("Serialized   : " + json);
        
        // Deserialize back
        Event parsed = mapper.readValue(json, Event.class);
        System.out.println("Deserialized : " + parsed);
    }
}
```
**Expected Output**:
```
Serialized   : {"eventName":"Deployment","timestamp":"2026-10-03T14:30:00"}
Deserialized : Event[eventName=Deployment, timestamp=2026-10-03T14:30]
```
**Why This Output**: `ParameterNamesModule` enables Record deserialization by reading constructor parameter names. `JavaTimeModule` handles `LocalDateTime` serialization. Disabling `WRITE_DATES_AS_TIMESTAMPS` produces ISO-8601 strings instead of numeric timestamps. The Record is serialized with its component names and deserialized back into a new Record instance.

### Real-World Cases
- **Spring Boot**: Auto-configures `ObjectMapper` with sensible defaults
- **Microservices**: Shared `ObjectMapper` singleton for consistent JSON handling
- **Configuration**: Load JSON config files into Records at startup

### References
- Jackson Databind GitHub - https://github.com/FasterXML/jackson-databind
- Baeldung: Intro to the Jackson ObjectMapper - https://www.baeldung.com/jackson-object-mapper-tutorial
- Baeldung: Jackson Date - https://www.baeldung.com/jackson-serialize-dates

---

## Core Concept 3: Serialization & Deserialization

### Definitions
**Core Definition**: Serialization is converting a Java object into a JSON string; deserialization is parsing a JSON string back into a Java object graph.

**Technical Definition**: Jackson's `ObjectMapper.writeValueAsString(Object)` serializes Java objects to JSON, while `readValue(String, Class<T>)` deserializes JSON into typed Java objects. For generic types, `TypeReference<T>` preserves type information. For nested records and collections, `TypeReference` is required to avoid type erasure.

**Beginner-Friendly Explanation**: Serialization is like writing a letter—you take your thoughts (Java object) and put them into a standard envelope (JSON). Deserialization is opening the envelope and reconstructing the thoughts (Java object). Jackson handles all the formatting and type conversion.

### Purposes
- To convert Java objects to JSON for network transmission
- To parse incoming JSON payloads into typed Java objects
- To persist object state as JSON text
- To support generic collections and nested types
- To handle polymorphic types with type information

### Syntax Rules and Structure

#### Complete General Syntax: Serialization & Deserialization
```
SERIALIZATION & DESERIALIZATION
│
├── Serialization
│   ├── String json = mapper.writeValueAsString(obj)
│   ├── byte[] bytes = mapper.writeValueAsBytes(obj)
│   └── mapper.writeValue(file, obj)
│
├── Deserialization (Simple)
│   └── T obj = mapper.readValue(json, T.class)
│
├── Deserialization (Generic)
│   └── List<T> list = mapper.readValue(json, new TypeReference<List<T>>() {})
│
└── Deserialization (Tree)
    └── JsonNode node = mapper.readTree(json)
```

#### Component Breakdown
| Operation | Method | Returns |
|-----------|--------|---------|
| Java → JSON String | `writeValueAsString()` | `String` |
| Java → JSON Bytes | `writeValueAsBytes()` | `byte[]` |
| Java → File | `writeValue(File, Object)` | void |
| JSON → Java | `readValue(String, Class)` | `T` |
| JSON → Generic | `readValue(String, TypeReference)` | `T` |
| JSON → Tree | `readTree(String)` | `JsonNode` |

#### Syntax Rules
- Use `TypeReference` for generic collections (`List<User>`, `Map<String, List<Order>>`)
- Use `@JsonCreator` and `@JsonProperty` for custom constructors
- Use `@JsonIgnore` to exclude fields from serialization
- Use `@JsonProperty` to rename fields during serialization/deserialization

#### Constraints and Limitations
- Type erasure requires `TypeReference` for generics
- Cyclic object graphs cause `StackOverflowError` (use `@JsonIdentityInfo`)
- No-arg constructors are required for POJOs (Records use canonical constructors)
- Large objects cause memory pressure (use streaming API)

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Serialization and Deserialization
```java
// SerializationDemo.java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.module.paramnames.ParameterNamesModule;
import java.util.List;

public class SerializationDemo {
    
    record Address(String street, String city) {}
    record User(int id, String name, List<Address> addresses) {}
    
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new ParameterNamesModule());
        
        // Create object graph
        User user = new User(1, "Alice", List.of(
            new Address("123 Main St", "Springfield"),
            new Address("456 Oak Ave", "Shelbyville")
        ));
        
        // Serialize
        String json = mapper.writeValueAsString(user);
        System.out.println("Serialized JSON:");
        System.out.println(json);
        
        // Deserialize back (with TypeReference for nested generics)
        User parsed = mapper.readValue(json, User.class);
        System.out.println("\nDeserialized User:");
        System.out.println("  ID: " + parsed.id());
        System.out.println("  Name: " + parsed.name());
        System.out.println("  Addresses:");
        parsed.addresses().forEach(a -> 
            System.out.println("    " + a.street() + ", " + a.city()));
    }
}
```
**Expected Output**:
```
Serialized JSON:
{"id":1,"name":"Alice","addresses":[{"street":"123 Main St","city":"Springfield"},{"street":"456 Oak Ave","city":"Shelbyville"}]}

Deserialized User:
  ID: 1
  Name: Alice
  Addresses:
    123 Main St, Springfield
    456 Oak Ave, Shelbyville
```
**Why This Output**: The Record is serialized with nested Address records as JSON arrays. Deserialization uses the canonical constructor (enabled by `ParameterNamesModule`). Nested generics work because `User` is a top-level class with a concrete `List<Address>` type—the `TypeReference` is not strictly needed here because `User.class` carries the generic signature.

### Real-World Cases
- **REST Clients**: `RestTemplate`/`WebClient` use Jackson for request/response bodies
- **Event Sourcing**: Events serialized as JSON for event stores
- **Caching**: Objects cached as JSON strings in Redis

### References
- Baeldung: Intro to the Jackson ObjectMapper - https://www.baeldung.com/jackson-object-mapper-tutorial
- Baeldung: Jackson - Marshall String to JsonNode - https://www.baeldung.com/jackson-json-node

---

## Core Concept 4: Modern DTOs with Records

### Definitions
**Core Definition**: Java Records are immutable, transparent data carriers that serve as concise DTOs, with Jackson 2.12+ providing native serialization and deserialization support.

**Technical Definition**: Java Records (finalized in Java 16) are classes that declare their state in the class header. The compiler generates a canonical constructor, accessor methods, `equals()`, `hashCode()`, and `toString()`. Jackson serializes Records by component order and deserializes by invoking the canonical constructor with JSON values mapped to parameter names. Jackson requires the `-parameters` compiler flag to retain parameter names; without it, deserialization fails with `InvalidDefinitionException`.

**Beginner-Friendly Explanation**: Records are Java's way of saying "this class is just data." Instead of writing 50 lines of boilerplate for a simple DTO, you write one line. Jackson understands Records natively, so you don't need getters, setters, or no-arg constructors. Just enable the `-parameters` flag and you're done.

### Purposes
- To eliminate DTO boilerplate (getters, setters, constructors, equals/hashCode)
- To provide immutable data carriers for thread-safe JSON processing
- To align with modern Java idioms (Java 16+)
- To improve code readability and maintainability
- To integrate with Spring Boot 3 / Spring 6 auto-configuration

### Syntax Rules and Structure

#### Complete General Syntax: Record with Jackson
```
RECORD WITH JACKSON
│
├── 1. Define Record
│   └── record User(int id, String name, String email) {}
│
├── 2. Maven Compiler Configuration
│   └── <parameters>true</parameters>  ← CRITICAL
│
├── 3. Register ParameterNamesModule
│   └── mapper.registerModule(new ParameterNamesModule())
│
├── 4. Serialize
│   └── {"id":1,"name":"Alice","email":"alice@example.com"}
│
└── 5. Deserialize
    └── mapper.readValue(json, User.class)
```

#### Component Breakdown
| Feature | Record | POJO |
|---------|--------|------|
| Boilerplate | Minimal | Extensive |
| Immutability | Built-in | Manual |
| Jackson Support | 2.12+ | All versions |
| `-parameters` Flag | Required | Not required |
| Canonical Constructor | Auto-generated | Manual |

#### Syntax Rules
- Add `-parameters` to `maven-compiler-plugin` configuration
- Register `ParameterNamesModule` on `ObjectMapper`
- Use `@JsonProperty` to rename JSON fields if parameter names don't match
- Use `@JsonCreator` for Records with multiple constructors
- Records cannot be JPA entities (no no-arg constructor, final fields)

#### Constraints and Limitations
- `-parameters` flag must be enabled or deserialization fails
- Records cannot be used with JPA/Hibernate as entities
- Records cannot be proxied by CGLIB (final class)
- Nested generics require `TypeReference` for deserialization

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Record Serialization/Deserialization
```java
// RecordDtoDemo.java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.module.paramnames.ParameterNamesModule;

public class RecordDtoDemo {
    
    record Product(String sku, String name, double price) {}
    
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new ParameterNamesModule());
        
        // Serialize Record
        Product product = new Product("SKU-001", "Laptop", 1299.99);
        String json = mapper.writeValueAsString(product);
        System.out.println("Serialized: " + json);
        
        // Deserialize back to Record
        Product parsed = mapper.readValue(json, Product.class);
        System.out.println("Deserialized: " + parsed);
        System.out.println("  SKU: " + parsed.sku());
        System.out.println("  Name: " + parsed.name());
        System.out.println("  Price: " + parsed.price());
    }
}
```
**Expected Output**:
```
Serialized: {"sku":"SKU-001","name":"Laptop","price":1299.99}
Deserialized: Product[sku=SKU-001, name=Laptop, price=1299.99]
  SKU: SKU-001
  Name: Laptop
  Price: 1299.99
```
**Why This Output**: Jackson serializes the Record's components as JSON fields. Deserialization uses the canonical constructor with parameter names retained by `-parameters`. The `ParameterNamesModule` enables Jackson to map JSON field names to constructor parameter names. Without the `-parameters` flag, Jackson would throw `InvalidDefinitionException`.

### Real-World Cases
- **Spring Boot 3**: Records as REST controller DTOs
- **Microservices**: Immutable request/response payloads
- **Configuration Properties**: `@ConfigurationProperties` with Records
- **Testing**: Record-based test fixtures

### References
- Jackson Java Records Support - DeepWiki - https://deepwiki.com/FasterXML/jackson-databind
- Java Record 终极权威指南 - Tencent Cloud - https://cloud.tencent.cn/developer/article/2719382
- Records as DTOs - Trinity Logic - https://www.trinitylogic.co.uk

---

## Core Concept 5: Customizing Mapping

### Definitions
**Core Definition**: Customizing mapping is the configuration of Jackson's behavior through naming strategies, unknown property handling, and date/time formatting to align JSON representation with application conventions.

**Technical Definition**: Jackson provides `PropertyNamingStrategies` for field name conversion (camelCase ↔ snake_case), `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES` for unknown field handling, and `@JsonFormat` for date/time patterns. Configuration can be global (on `ObjectMapper`) or per-class (via annotations).

**Beginner-Friendly Explanation**: Sometimes JSON field names don't match Java naming conventions. A REST API might use `user_name` while your Java Record uses `userName`. Jackson's naming strategies automatically translate between the two. Similarly, you can tell Jackson to ignore extra fields in JSON or format dates in a specific pattern.

### Purposes
- To align JSON field names with API conventions (snake_case, kebab-case)
- To handle evolving APIs that add new fields gracefully
- To format dates consistently across serialization and deserialization
- To rename specific fields with `@JsonProperty`
- To exclude sensitive fields with `@JsonIgnore`

### Syntax Rules and Structure

#### Complete General Syntax: Mapping Customization
```
MAPPING CUSTOMIZATION
│
├── Naming Strategy (Global)
│   └── mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)
│
├── Naming Strategy (Per-Class)
│   └── @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
│
├── Ignore Unknown Properties (Global)
│   └── mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
│
├── Ignore Unknown Properties (Per-Class)
│   └── @JsonIgnoreProperties(ignoreUnknown = true)
│
├── Rename Field
│   └── @JsonProperty("custom_name") private String name;
│
├── Exclude Field
│   └── @JsonIgnore private String password;
│
└── Date Formatting
    └── @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "UTC")
```

#### Component Breakdown
| Strategy | Constant | Example |
|----------|----------|---------|
| Camel Case | `LOWER_CAMEL_CASE` | `userName` |
| Snake Case | `SNAKE_CASE` | `user_name` |
| Kebab Case | `KEBAB_CASE` | `user-name` |
| Lower Case | `LOWER_CASE` | `username` |
| Lower Dot | `LOWER_DOT_CASE` | `user.name` |

#### Syntax Rules
- `PropertyNamingStrategies` (Jackson 2.12+) replaces deprecated `PropertyNamingStrategy`
- `FAIL_ON_UNKNOWN_PROPERTIES` defaults to `true`; set to `false` to ignore unknown fields
- `@JsonIgnoreProperties(ignoreUnknown = true)` is per-class only
- `@JsonFormat` overrides default date formatting
- In Jackson 3.0, `WRITE_DATES_AS_TIMESTAMPS` defaults to `false`

#### Constraints and Limitations
- Global naming strategy affects all classes
- `@JsonFormat` requires `jackson-datatype-jsr310` for Java 8 Date/Time
- Per-class annotations override global configuration
- Snake case strategy does not handle `is_active` ↔ `isActive` perfectly

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Naming Strategy and Date Formatting
```java
// MappingCustomizationDemo.java
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.module.paramnames.ParameterNamesModule;
import java.time.LocalDateTime;

public class MappingCustomizationDemo {
    
    record UserProfile(
        int userId,
        String userName,
        @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "UTC")
        LocalDateTime createdAt
    ) {}
    
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new ParameterNamesModule());
        mapper.registerModule(new JavaTimeModule());
        
        // Apply snake_case naming strategy
        mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);
        
        // Ignore unknown properties
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        
        // Serialize with snake_case and formatted date
        UserProfile profile = new UserProfile(
            1, "Alice", LocalDateTime.of(2026, 10, 3, 14, 30, 0));
        String json = mapper.writeValueAsString(profile);
        System.out.println("Serialized (snake_case, formatted date):");
        System.out.println(json);
        
        // Deserialize JSON with extra unknown field
        String inputJson = """
            {
                "user_id": 2,
                "user_name": "Bob",
                "created_at": "2026-10-03 15:45:00",
                "extra_field": "ignored"
            }
            """;
        UserProfile parsed = mapper.readValue(inputJson, UserProfile.class);
        System.out.println("\nDeserialized (unknown field ignored):");
        System.out.println("  ID: " + parsed.userId());
        System.out.println("  Name: " + parsed.userName());
        System.out.println("  Created: " + parsed.createdAt());
    }
}
```
**Expected Output**:
```
Serialized (snake_case, formatted date):
{"user_id":1,"user_name":"Alice","created_at":"2026-10-03 14:30:00"}

Deserialized (unknown field ignored):
  ID: 2
  Name: Bob
  Created: 2026-10-03T15:45
```
**Why This Output**: The `SNAKE_CASE` naming strategy converts `userId` → `user_id`, `userName` → `user_name`, and `createdAt` → `created_at`. The `@JsonFormat` pattern formats the `LocalDateTime` as `yyyy-MM-dd HH:mm:ss`. `FAIL_ON_UNKNOWN_PROPERTIES=false` allows the extra `extra_field` to be ignored during deserialization.

### Real-World Cases
- **REST APIs**: Snake case for JSON, camel case for Java
- **Third-Party APIs**: Ignoring unknown fields for forward compatibility
- **Date/Time**: Consistent ISO-8601 or custom patterns across services
- **Security**: `@JsonIgnore` for passwords and sensitive tokens

### References
- PropertyNamingStrategies Javadoc - https://adobedocs.github.io/aem-developer-materials/
- Baeldung: Handling Unknown Properties - https://www.baeldung.com/members/courses/learn-json-with-jackson/
- Baeldung: Jackson Date - https://www.baeldung.com/jackson-serialize-dates

---

## Core Concept 6: Tree Model & Streaming

### Definitions
**Core Definition**: The tree model (`JsonNode`) provides a flexible in-memory representation of JSON for dynamic navigation, while the streaming API (`JsonParser`) processes JSON token-by-token for memory-efficient handling of large payloads.

**Technical Definition**: `ObjectMapper.readTree(String)` parses JSON into a `JsonNode` tree, allowing navigation via `get()`, `path()`, and `findPath()`. The streaming API uses `JsonParser` to iterate over JSON tokens (`START_OBJECT`, `FIELD_NAME`, `VALUE_STRING`, etc.) without building an object tree, reducing memory overhead for large payloads. The tree model is suitable for dynamic schemas; the streaming API is suitable for large files or streams.

**Beginner-Friendly Explanation**: The tree model is like reading a book and memorizing its structure so you can jump to any chapter. The streaming API is like reading the book page by page, processing each word as it comes—faster and uses less memory, but you can't jump backward. Use the tree model when you need to explore the JSON structure; use streaming when you're processing a huge file and only need specific fields.

### Purposes
- To parse JSON with unknown or dynamic structure
- To navigate nested JSON without defining Java classes
- To process large JSON streams with minimal memory
- To extract specific fields from large payloads efficiently
- To modify JSON trees programmatically

### Syntax Rules and Structure

#### Complete General Syntax: Tree Model & Streaming
```
TREE MODEL & STREAMING
│
├── Tree Model (JsonNode)
│   ├── JsonNode root = mapper.readTree(json)
│   ├── JsonNode field = root.get("fieldName")
│   ├── String text = field.asText()
│   ├── int num = field.asInt()
│   └── JsonNode nested = root.path("a").path("b")
│
├── Streaming API (JsonParser)
│   ├── JsonParser parser = mapper.createParser(input)
│   ├── while (parser.nextToken() != null) { ... }
│   ├── JsonToken token = parser.currentToken()
│   └── String value = parser.getText()
│
└── Hybrid
    ├── Read tree for structure
    └── Stream for large arrays
```

#### Component Breakdown
| Approach | Memory | Speed | Use Case |
|----------|--------|-------|----------|
| Tree Model | High | Moderate | Dynamic schemas |
| Streaming | Low | Fast | Large files |
| Data Binding | Moderate | Fast | Known schemas |

#### Syntax Rules
- Use `readTree()` for dynamic JSON exploration
- Use `path()` instead of `get()` to avoid `NullPointerException`
- Use `asText()`, `asInt()`, `asBoolean()` for type conversion
- Use `createParser()` for streaming; call `nextToken()` to iterate
- Close `JsonParser` after use (try-with-resources)

#### Constraints and Limitations
- Tree model loads entire JSON into memory
- Streaming API is forward-only (no random access)
- Tree model is slower for large payloads
- Streaming requires manual token handling

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Tree Model with JsonNode
```java
// TreeModelDemo.java
import com.fasterxml.jackson.databind.*;

public class TreeModelDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        
        String json = """
            {
                "user": {
                    "id": 1,
                    "name": "Alice",
                    "roles": ["ADMIN", "USER"]
                },
                "metadata": {
                    "version": 2
                }
            }
            """;
        
        // Parse into tree
        JsonNode root = mapper.readTree(json);
        
        // Navigate with path (null-safe)
        String userName = root.path("user").path("name").asText();
        int userId = root.path("user").path("id").asInt();
        
        System.out.println("User: " + userName + " (ID: " + userId + ")");
        
        // Iterate array
        JsonNode roles = root.path("user").path("roles");
        System.out.println("Roles:");
        roles.forEach(role -> System.out.println("  - " + role.asText()));
        
        // Check existence
        boolean hasMetadata = root.has("metadata");
        System.out.println("Has metadata: " + hasMetadata);
        
        // Build a new tree
        ObjectNode newNode = mapper.createObjectNode();
        newNode.put("status", "processed");
        newNode.put("userName", userName);
        System.out.println("\nBuilt node: " + mapper.writeValueAsString(newNode));
    }
}
```
**Expected Output**:
```
User: Alice (ID: 1)
Roles:
  - ADMIN
  - USER
Has metadata: true

Built node: {"status":"processed","userName":"Alice"}
```
**Why This Output**: `readTree()` parses JSON into a navigable tree. `path()` is null-safe (returns `MissingNode` instead of throwing). `asText()` and `asInt()` convert values. `forEach()` iterates array elements. `createObjectNode()` builds a new JSON structure programmatically.

---

#### Example 2: Streaming API with JsonParser
```java
// StreamingApiDemo.java
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.StringReader;

public class StreamingApiDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        
        String json = """
            {
                "records": [
                    {"id": 1, "name": "Alice"},
                    {"id": 2, "name": "Bob"},
                    {"id": 3, "name": "Charlie"}
                ]
            }
            """;
        
        // Stream through JSON tokens
        try (JsonParser parser = mapper.createParser(new StringReader(json))) {
            JsonToken token;
            int recordCount = 0;
            String currentName = null;
            
            while ((token = parser.nextToken()) != null) {
                if (token == JsonToken.FIELD_NAME) {
                    currentName = parser.currentName();
                } else if (token == JsonToken.VALUE_STRING && "name".equals(currentName)) {
                    recordCount++;
                    System.out.println("Record " + recordCount + 
                        " name: " + parser.getText());
                }
            }
            
            System.out.println("\nTotal records: " + recordCount);
        }
    }
}
```
**Expected Output**:
```
Record 1 name: Alice
Record 2 name: Bob
Record 3 name: Charlie

Total records: 3
```
**Why This Output**: `createParser()` returns a `JsonParser` that streams tokens. `nextToken()` advances to the next token. `FIELD_NAME` tokens provide the current field name; `VALUE_STRING` tokens provide string values. This approach processes JSON without building an object tree, making it memory-efficient for large payloads.

### Real-World Cases
- **Webhooks**: Processing variable JSON payloads with `JsonNode`
- **Log Processing**: Streaming large JSON logs with `JsonParser`
- **API Gateways**: Extracting specific fields from large responses
- **Configuration**: Dynamic configuration with unknown keys

### References
- Baeldung: Working with Tree Model Nodes in Jackson - https://www.baeldung.com/jackson-json-node
- Baeldung: Mapping a Dynamic JSON Object with Jackson - https://www.baeldung.com/jackson-dynamic-json
- Stack Overflow: Jackson Streaming API - https://stackoverflow.com

---

## Deprecation and Safety Notes

| Feature | Status | Notes |
|---------|--------|-------|
| `PropertyNamingStrategy` | Deprecated (2.12) | Use `PropertyNamingStrategies` |
| `WRITE_DATES_AS_TIMESTAMPS` | Default `true` (2.x) | Default `false` in Jackson 3.0 |
| `FAIL_ON_UNKNOWN_PROPERTIES` | Default `true` | Set to `false` for evolving APIs |
| `-parameters` compiler flag | Required for Records | Without it, `InvalidDefinitionException` |
| `JsonParser` | Active | Use try-with-resources for cleanup |
| `JsonNode` | Active | Loads entire tree into memory |
| Records as JPA Entities | Not supported | Use as DTO projections only |
| CGLIB Proxying of Records | Not supported | Records are final classes |

---

## References

### Official Documentation
- Jackson Databind GitHub Repository - https://github.com/FasterXML/jackson-databind
- Jackson Core Javadoc - https://fasterxml.github.io/jackson-core/
- PropertyNamingStrategies Javadoc - https://adobedocs.github.io/aem-developer-materials/

### Tutorials and Guides
- Baeldung: Intro to the Jackson ObjectMapper - https://www.baeldung.com/jackson-object-mapper-tutorial
- Baeldung: Jackson Date - https://www.baeldung.com/jackson-serialize-dates
- Baeldung: Working with Tree Model Nodes in Jackson - https://www.baeldung.com/jackson-json-node
- Baeldung: Mapping a Dynamic JSON Object with Jackson - https://www.baeldung.com/jackson-dynamic-json
- Baeldung: Handling Unknown Properties - https://www.baeldung.com/members/courses/learn-json-with-jackson/

### Records and Jackson
- Jackson Java Records Support - DeepWiki - https://deepwiki.com/FasterXML/jackson-databind
- Java Record 终极权威指南 - Tencent Cloud - https://cloud.tencent.cn/developer/article/2719382
- Records as DTOs - Trinity Logic - https://www.trinitylogic.co.uk

### Naming and Formatting
- Jackson PropertyNamingStrategies - Adobe AEM Javadoc - https://adobedocs.github.io/aem-developer-materials/
- Jackson Date Formatting - OpenRewrite - https://docs.openrewrite.org
- Stack Overflow: Jackson Streaming API - https://stackoverflow.com

### JSON Processing
- Apache Hive JsonReader - Apache SVN - https://svn.apache.org/repos/
- Eclipse JSON Class - Eclipse - https://eclipse.dev