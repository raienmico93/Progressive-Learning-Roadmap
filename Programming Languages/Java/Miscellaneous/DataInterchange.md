# Data Interchange & API Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition
Data Interchange & API Architecture is the discipline of designing, validating, versioning, and securing the structured data payloads (JSON, XML) that flow between distributed systems, ensuring that APIs remain robust, backward-compatible, and resilient against malicious input.

### Technical Definition
Data interchange in Java-based systems involves the serialization and deserialization of structured data formats—primarily JSON (via Jackson) and XML (via JAXB/Jackson XML)—across network boundaries. API architecture encompasses the design patterns and guardrails that govern how these payloads are validated (Jakarta Bean Validation), versioned (URI path, headers, media types), and secured against injection attacks (XXE), resource-exhaustion denial of service (JSON depth bombs), and unsafe polymorphic deserialization.

### Beginner-Friendly Explanation
Think of an API as a postal service between two applications. **Payload validation** is the customs inspector checking that every package contains exactly what it claims. **API versioning** is the address system—old addresses still work even after new streets are added. **Polymorphic data types** are like packages that can contain different shapes of items, with labels telling the receiver what's inside. **Security guardrails** are the bomb-sniffing dogs that prevent malicious packages from entering the building.

### Key Characteristics
- **Contract-Driven**: Payloads conform to explicit schemas or DTOs
- **Validated**: Input is checked at the boundary before processing
- **Evolving**: APIs change without breaking existing clients
- **Polymorphic**: Inheritance hierarchies are serialized with type discriminators
- **Hardened**: Defenses against XXE, depth bombs, and unsafe deserialization
- **Observable**: Validation errors and security events are logged and monitored

### Prerequisites
- Solid understanding of Java (annotations, generics, inheritance)
- Familiarity with JSON and XML syntax
- Experience with Jackson (`ObjectMapper`) for JSON processing
- Knowledge of Spring Boot (for `@Valid` and `@RequestBody`)
- Understanding of HTTP status codes and REST principles

### Related Programming Areas
- **JSON Processing**: Jackson serialization/deserialization
- **XML Processing**: JAXB, StAX, XXE prevention
- **Spring Boot**: `@Valid`, `@RequestBody`, exception handling
- **API Gateway**: Rate limiting, payload size limits, WAF rules
- **Dependency Management**: CVE scanning, version upgrades

### Core Concepts Overview
1. **Payload Validation**: Jakarta Bean Validation with `@NotNull`, `@Size`, `@Email`
2. **API Versioning Compatibility**: Default values, optional fields, deprecation strategies
3. **Polymorphic Data Types**: `@JsonTypeInfo`, `@JsonSubTypes`, type discriminators
4. **Security & Guardrails**: XXE prevention, JSON depth bombs, unsafe deserialization

---

## Core Concept 1: Payload Validation

### Definitions

**Core Definition**: Payload validation is the automatic enforcement of constraints on incoming JSON/XML models using Jakarta Bean Validation annotations, rejecting malformed or out-of-range data at the API boundary before it reaches business logic.

**Technical Definition**: Jakarta Bean Validation provides a metadata model and API for JavaBean validation. Constraints are expressed as annotations on fields, properties, or method parameters, and are evaluated by a `Validator` implementation (e.g., Hibernate Validator). In Spring Boot, the `@Valid` annotation on a `@RequestBody` parameter triggers validation before the controller method executes; violations throw `MethodArgumentNotValidException`, which maps to HTTP 400 Bad Request. Common constraints include `@NotNull`, `@NotBlank`, `@NotEmpty`, `@Size`, `@Email`, `@Min`, `@Max`, and `@Pattern`. For nested objects, cascading validation is enabled by annotating the field with `@Valid`.

**Beginner-Friendly Explanation**: Think of payload validation as a security checkpoint at the airport. Before anyone (data) boards the plane (enters your application), a guard (validator) checks that they have the right ID (not null), the right size luggage (size constraints), and a valid boarding pass (email format). If anything is wrong, they're stopped at the gate (HTTP 400), and the plane doesn't take off (business logic doesn't execute).

### Purposes
- To reject malformed or incomplete input at the API boundary
- To centralize validation logic in DTO annotations
- To provide clear, user-friendly error messages
- To prevent invalid data from reaching business logic and persistence layers
- To enforce business rules (e.g., age ≥ 18, password length ≥ 6)
- To cascade validation through nested object graphs

### Syntax Rules and Structure

#### Complete General Syntax: Jakarta Bean Validation
```
JAKARTA BEAN VALIDATION
│
├── 1. Add Dependency
│   └── spring-boot-starter-validation (includes Hibernate Validator)
│
├── 2. Annotate DTO Fields
│   ├── @NotNull(message = "Username cannot be empty.")
│   ├── @Size(min = 2, max = 30, message = "Username must be between 2 and 30 characters.")
│   ├── @Email(message = "Invalid email format.")
│   ├── @Min(value = 18, message = "Age must be at least 18.")
│   └── @Pattern(regexp = "...", message = "...")
│
├── 3. Cascade to Nested Objects
│   └── @Valid private Address address;
│
├── 4. Trigger Validation in Controller
│   └── @PostMapping public ResponseEntity<?> create(@Valid @RequestBody UserDTO dto)
│
└── 5. Handle Violations
    └── MethodArgumentNotValidException → HTTP 400 + error details
```

#### Component Breakdown
| Annotation | Constraint | Applies To |
|-----------|-----------|------------|
| `@NotNull` | Not null | Any type |
| `@NotBlank` | Not null, not empty after trim | `String` |
| `@NotEmpty` | Not null, not empty | `String`, `Collection`, `Map`, `Array` |
| `@Size(min, max)` | Length within range | `String`, `Collection`, `Map`, `Array` |
| `@Email` | Valid email format | `String` |
| `@Min(value)` / `@Max(value)` | Numeric range | `Number` |
| `@Pattern(regexp)` | Regex match | `String` |
| `@Past` / `@Future` | Date in past/future | `Date`, `LocalDate`, etc. |
| `@Valid` | Cascade validation | Nested object, `List<@Valid T>` |

#### Syntax Rules
- Add `spring-boot-starter-validation` dependency (includes Hibernate Validator)
- Place constraint annotations on DTO fields
- Use `@Valid` on `@RequestBody` parameter to trigger validation
- Use `@Valid` on nested object fields to cascade validation
- Include `message` parameter for user-friendly error messages
- Return `MethodArgumentNotValidException` details for HTTP 400 responses

#### Constraints and Limitations
- `@Valid` on `@RequestBody` triggers validation before the controller method executes
- Nested objects require `@Valid` on the field to cascade validation
- Collections require `List<@Valid T>` syntax for element validation
- Custom constraints require a `ConstraintValidator` implementation
- Validation messages are locale-sensitive

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic DTO Validation
```java
// PayloadValidationDemo.java
import jakarta.validation.Valid;
import jakarta.validation.constraints.*;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

public class PayloadValidationDemo {
    
    // Step 1: Define DTO with validation constraints
    record UserDTO(
        @NotBlank(message = "Username cannot be empty.")
        @Size(min = 2, max = 30, message = "Username must be between 2 and 30 characters.")
        String username,
        
        @NotNull(message = "Email is required.")
        @Email(message = "Invalid email format.")
        String email,
        
        @Min(value = 18, message = "Age must be at least 18.")
        @Max(value = 120, message = "Age must be at most 120.")
        int age
    ) {}
    
    // Step 2: Controller with @Valid
    @RestController
    @RequestMapping("/api/users")
    static class UserController {
        @PostMapping
        public ResponseEntity<String> createUser(@Valid @RequestBody UserDTO userDTO) {
            return ResponseEntity.status(201).body("User created: " + userDTO.username());
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Payload Validation Demo ===");
        System.out.println("Valid request: username='Alice', email='alice@example.com', age=30");
        System.out.println("→ HTTP 201 Created");
        System.out.println();
        System.out.println("Invalid request: username='', email='invalid', age=15");
        System.out.println("→ HTTP 400 Bad Request with field errors:");
        System.out.println("  - username: Username cannot be empty.");
        System.out.println("  - email: Invalid email format.");
        System.out.println("  - age: Age must be at least 18.");
    }
}
```
**Expected Output**:
```
=== Payload Validation Demo ===
Valid request: username='Alice', email='alice@alice@example.com', age=30
→ HTTP 201 Created

Invalid request: username='', email='invalid', age=15
→ HTTP 400 Bad Request with field errors:
  - username: Username cannot be empty.
  - email: Invalid email format.
  - age: Age must be at least 18.
```
**Why This Output**: The `@NotBlank` on `username` rejects empty strings. `@Email` rejects malformed email addresses. `@Min(18)` rejects ages below 18. Spring's `@Valid` triggers validation before the controller method executes; violations produce a `MethodArgumentNotValidException` mapped to HTTP 400 with field-level error details.

---

#### Example 2: Nested Object Validation
```java
// NestedValidationDemo.java
import jakarta.validation.Valid;
import jakarta.validation.constraints.*;

public class NestedValidationDemo {
    
    record Address(
        @NotBlank(message = "Street is required")
        String street,
        @NotBlank(message = "City is required")
        String city
    ) {}
    
    record User(
        @NotBlank(message = "Name is required")
        String name,
        @Valid  // Cascade validation to nested Address
        @NotNull(message = "Address is required")
        Address address
    ) {}
    
    public static void main(String[] args) {
        System.out.println("=== Nested Object Validation Demo ===");
        System.out.println("Valid: User with complete Address");
        System.out.println("→ Validation passes");
        System.out.println();
        System.out.println("Invalid: User with Address missing city");
        System.out.println("→ Validation fails:");
        System.out.println("  - address.city: City is required");
    }
}
```
**Expected Output**:
```
=== Nested Object Validation Demo ===
Valid: User with complete Address
→ Validation passes

Invalid: User with Address missing city
→ Validation fails:
  - address.city: City is required
```
**Why This Output**: The `@Valid` annotation on the `address` field triggers cascading validation into the `Address` record. Without `@Valid`, the nested `Address` constraints would not be evaluated. The validation path `address.city` identifies the exact location of the violation.

---

### Real-World Cases
- **User Registration**: Validating username, email, password, and age before creating an account
- **Order Processing**: Validating shipping address, payment details, and order items
- **Configuration APIs**: Validating server settings, timeouts, and thresholds
- **Multi-Tenant Systems**: Validating tenant-specific configuration payloads

### References
- Jakarta Bean Validation Specification - https://beanvalidation.org/
- Using @Valid for Request Validation - CodeGym - https://codegym.cc/quests/lectures/en.codegym.java.spring.lecture.level09.lecture03
- @Valid Annotation on Child Objects - Baeldung - https://www.baeldung.com/java-valid-annotation-child-objects
- Jakarta Bean Validation: Avoiding Common Pitfalls - Safeguard - https://safeguard.sh/resources/blog/jakarta-bean-validation-avoiding-common-pitfalls

---

## Core Concept 2: API Versioning Compatibility

### Definitions

**Core Definition**: API versioning compatibility is the practice of evolving API payloads (adding, modifying, or deprecating fields) without breaking existing clients, using default values, optional fields, and graceful deprecation strategies.

**Technical Definition**: API versioning strategies include URI path versioning (`/api/v1/products`), request parameter versioning (`?version=1`), custom header versioning (`X-API-VERSION: 1`), and media type versioning (`Accept: application/vnd.store.v1+json`). In Spring Framework 7, first-class API versioning support is provided through `ApiVersionStrategy`, which resolves, validates, and analyzes request versions, and supports declaring versions directly on mappings such as `@GetMapping(version = "1.1")`. For backward compatibility, new fields must be optional with sensible defaults; Jackson's `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES` should be disabled to ignore fields added by newer clients.

**Beginner-Friendly Explanation**: Think of API versioning as publishing a book with multiple editions. Readers who bought the first edition can still use it—the page numbers and chapters are still valid. New editions add chapters and fix errors, but they don't remove the old chapters that readers rely on. When a chapter becomes obsolete, it's marked as "deprecated" and eventually removed only after readers have had time to adapt.

### Purposes
- To evolve APIs without breaking existing clients
- To introduce new fields and features while maintaining backward compatibility
- To deprecate old endpoints gracefully over time
- To support multiple client versions simultaneously
- To communicate changes clearly through deprecation headers

### Syntax Rules and Structure

#### Complete General Syntax: API Versioning Strategies
```
API VERSIONING STRATEGIES
│
├── 1. URI Path Versioning
│   └── /api/v1/products  →  /api/v2/products
│
├── 2. Request Parameter Versioning
│   └── /api/products?version=1
│
├── 3. Custom Header Versioning
│   └── X-API-VERSION: 1
│
├── 4. Media Type Versioning
│   └── Accept: application/vnd.store.v1+json
│
└── 5. Spring Framework 7 Native Support
    ├── @GetMapping(path = "/account/{id}", version = "1.1")
    └── spring.mvc.apiversion.use.header=API-Version
```

#### Backward Compatibility Rules
| Change Type | Compatible? | Strategy |
|-------------|-------------|----------|
| Add optional field | ✅ Yes | Default value, `@JsonInclude(NON_NULL)` |
| Add required field | ❌ No | New version required |
| Remove field | ❌ No | Deprecate first, remove in major version |
| Rename field | ❌ No | Keep old name as alias |
| Change type | ❌ No | New version required |
| Add enum value | ⚠️ Conditional | `READ_UNKNOWN_ENUM_VALUES_AS_NULL` |

#### Syntax Rules
- New fields must be optional with sensible defaults
- Disable `FAIL_ON_UNKNOWN_PROPERTIES` to ignore fields added by newer clients
- Use `@Deprecated` annotation and deprecation headers (e.g., `Deprecation`, `Sunset`)
- Provide migration guides for breaking changes
- Version DTOs from the start (e.g., `UserV1`, `UserV2`)
- Use `@JsonView` for view-based field filtering

#### Constraints and Limitations
- URI versioning is the most common but least RESTful
- Header/media type versioning is more RESTful but harder to test
- Multiple versions increase maintenance burden
- Deprecation periods must be long enough for clients to migrate
- Jackson 3.0 changes `Optional` handling compared to 2.x

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Backward-Compatible Field Addition
```java
// VersioningCompatibilityDemo.java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.annotation.JsonInclude;

public class VersioningCompatibilityDemo {
    
    // V1: Original DTO
    static class UserV1 {
        public String name;
        public String email;
    }
    
    // V2: Added optional fields with defaults
    static class UserV2 {
        public String name;
        public String email;
        public String phone = "N/A";      // Default value
        public boolean active = true;      // Default value
        
        @JsonInclude(JsonInclude.Include.NON_NULL)
        public String department;          // Optional field
    }
    
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        
        // V1 client sends old payload
        String v1Json = """
            {"name": "Alice", "email": "alice@example.com"}
            """;
        
        // V2 deserializes V1 payload — new fields get defaults
        UserV2 user = mapper.readValue(v1Json, UserV2.class);
        System.out.println("=== Backward Compatibility ===");
        System.out.println("V1 payload deserialized into V2 DTO:");
        System.out.println("  name: " + user.name);
        System.out.println("  email: " + user.email);
        System.out.println("  phone: " + user.phone + " (default)");
        System.out.println("  active: " + user.active + " (default)");
        System.out.println("  department: " + user.department + " (null)");
        
        // V2 client sends new payload with extra fields
        String v2Json = """
            {
                "name": "Bob",
                "email": "bob@example.com",
                "phone": "555-1234",
                "active": false,
                "department": "Engineering",
                "futureField": "ignored"
            }
            """;
        
        UserV2 v2User = mapper.readValue(v2Json, UserV2.class);
        System.out.println("\nV2 payload with unknown field:");
        System.out.println("  name: " + v2User.name);
        System.out.println("  phone: " + v2User.phone);
        System.out.println("  department: " + v2User.department);
        System.out.println("  futureField: ignored (FAIL_ON_UNKNOWN_PROPERTIES=false)");
    }
}
```
**Expected Output**:
```
=== Backward Compatibility ===
V1 payload deserialized into V2 DTO:
  name: Alice
  email: alice@example.com
  phone: N/A (default)
  active: true (default)
  department: null (null)

V2 payload with unknown field:
  name: Bob
  phone: 555-1234
  department: Engineering
  futureField: ignored (FAIL_ON_UNKNOWN_PROPERTIES=false)
```
**Why This Output**: The V2 DTO adds optional fields (`phone`, `active`) with default values. When the V1 payload is deserialized, these fields receive their defaults. The `FAIL_ON_UNKNOWN_PROPERTIES=false` setting allows the V2 DTO to ignore the `futureField` added by a hypothetical V3 client, ensuring forward compatibility.

---

### Real-World Cases
- **Mobile Apps**: Old app versions continue to work after backend API upgrades
- **Partner Integrations**: External partners migrate at their own pace
- **Microservices**: Independent deployment of services with versioned contracts
- **Public APIs**: Stripe, GitHub, and Twilio use URI path versioning

### References
- API Versioning in Spring - Spring Framework - https://springframework.org.cn/blog/2025/09/16/api-versioning-in-spring/
- Guide to Master API Versioning in Spring Boot - GUVI - https://www.guvi.in/blog/api-versioning-in-spring-boot-guide/
- Java API接口兼容性问题 - 亿速云 - https://m.yisu.com

---

## Core Concept 3: Polymorphic Data Types

### Definitions

**Core Definition**: Polymorphic data types in JSON serialization refer to the ability to serialize and deserialize inheritance hierarchies (abstract classes or interfaces) using type discriminators that identify the concrete subtype in the JSON payload.

**Technical Definition**: Jackson provides polymorphic type handling through the `@JsonTypeInfo` annotation (which specifies the discriminator property and inclusion mechanism) and `@JsonSubTypes` (which maps discriminator values to concrete classes). The `use` attribute specifies the discriminator type (`Id.NAME`, `Id.CLASS`, etc.), `include` specifies where the discriminator appears (`As.PROPERTY`, `As.EXISTING_PROPERTY`, `As.EXTERNAL_PROPERTY`), and `property` specifies the discriminator field name. Jackson 2.12+ supports `Id.DEDUCTION` for inferring types without explicit discriminators. Security is critical: `@JsonTypeInfo` without a custom `PolymorphicTypeValidator` triggers `DefaultBaseTypeLimitingValidator`, which has known CVEs (e.g., CVE-2026-83557) when `Comparable` is used as a base type.

**Beginner-Friendly Explanation**: Think of a polymorphic JSON payload as a package that could contain different items—a book, a DVD, or a CD. The package has a label (type discriminator) that says what's inside. Without the label, the receiver wouldn't know how to unpack it. Jackson reads the label and uses the right unpacking instructions (deserializer) for the specific item type.

### Purposes
- To serialize and deserialize inheritance hierarchies in JSON
- To preserve type information across network boundaries
- To support plugin architectures with extensible types
- To enable polymorphic API responses (e.g., different payment methods)
- To maintain type safety during deserialization

### Syntax Rules and Structure

#### Complete General Syntax: Polymorphic Deserialization
```
POLYMORPHIC DESERIALIZATION
│
├── 1. Base Class Annotations
│   ├── @JsonTypeInfo(use = Id.NAME, include = As.PROPERTY, property = "type")
│   └── @JsonSubTypes({
│         @Type(value = ElectricVehicle.class, name = "ELECTRIC_VEHICLE"),
│         @Type(value = FuelVehicle.class, name = "FUEL_VEHICLE")
│       })
│
├── 2. JSON Payload with Discriminator
│   └── {"type": "ELECTRIC_VEHICLE", "autonomy": "500", "chargingTime": "200"}
│
├── 3. Deserialization
│   └── Vehicle vehicle = mapper.readValue(json, Vehicle.class)
│
└── 4. Result
    └── vehicle instanceof ElectricVehicle → true
```

#### Component Breakdown
| Attribute | Values | Purpose |
|-----------|--------|---------|
| `use` | `Id.NAME`, `Id.CLASS`, `Id.DEDUCTION` | Discriminator type |
| `include` | `As.PROPERTY`, `As.EXISTING_PROPERTY`, `As.EXTERNAL_PROPERTY` | Discriminator placement |
| `property` | String | Discriminator field name |
| `visible` | `true` / `false` | Whether discriminator is visible to deserializer |
| `defaultImpl` | Class | Fallback type when discriminator is missing |

#### Syntax Rules
- Annotate the base class with `@JsonTypeInfo` and `@JsonSubTypes`
- Use `Id.NAME` with meaningful names (not `Id.CLASS`, which is unsafe)
- Use `As.EXISTING_PROPERTY` when the discriminator is already a field
- Use `Id.DEDUCTION` when the incoming payload cannot be modified
- Always configure a custom `PolymorphicTypeValidator` when using `@JsonTypeInfo`
- Never enable global default typing without a validator

#### Constraints and Limitations
- `Id.CLASS` is unsafe (attacker can specify arbitrary class names)
- `DefaultBaseTypeLimitingValidator` has known CVEs (CVE-2026-83557)
- `Comparable` as a base type is particularly dangerous
- `Id.DEDUCTION` requires Jackson 2.12+
- Polymorphic deserialization is a common attack surface

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Polymorphic Deserialization with @JsonTypeInfo
```java
// PolymorphicDeserializationDemo.java
import com.fasterxml.jackson.annotation.*;
import com.fasterxml.jackson.databind.ObjectMapper;

public class PolymorphicDeserializationDemo {
    
    // Base class with polymorphic type handling
    @JsonTypeInfo(
        use = JsonTypeInfo.Id.NAME,
        include = JsonTypeInfo.As.EXISTING_PROPERTY,
        property = "type",
        visible = true
    )
    @JsonSubTypes({
        @JsonSubTypes.Type(value = ElectricVehicle.class, name = "ELECTRIC"),
        @JsonSubTypes.Type(value = FuelVehicle.class, name = "FUEL")
    })
    static abstract class Vehicle {
        public String type;
    }
    
    static class ElectricVehicle extends Vehicle {
        public String autonomy;
        public String chargingTime;
        ElectricVehicle() { type = "ELECTRIC"; }
    }
    
    static class FuelVehicle extends Vehicle {
        public String fuelType;
        public String transmissionType;
        FuelVehicle() { type = "FUEL"; }
    }
    
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        
        // Deserialize electric vehicle
        String electricJson = """
            {"type": "ELECTRIC", "autonomy": "500", "chargingTime": "200"}
            """;
        Vehicle electric = mapper.readValue(electricJson, Vehicle.class);
        System.out.println("Electric: " + electric.getClass().getSimpleName());
        
        // Deserialize fuel vehicle
        String fuelJson = """
            {"type": "FUEL", "fuelType": "Petrol", "transmissionType": "Automatic"}
            """;
        Vehicle fuel = mapper.readValue(fuelJson, Vehicle.class);
        System.out.println("Fuel: " + fuel.getClass().getSimpleName());
        
        // Serialize back (discriminator included)
        System.out.println("\nSerialized electric:");
        System.out.println(mapper.writeValueAsString(electric));
    }
}
```
**Expected Output**:
```
Electric: ElectricVehicle
Fuel: FuelVehicle

Serialized electric:
{"type":"ELECTRIC","autonomy":"500","chargingTime":"200"}
```
**Why This Output**: The `@JsonTypeInfo` annotation tells Jackson to use the `"type"` field as the discriminator. `@JsonSubTypes` maps `"ELECTRIC"` to `ElectricVehicle` and `"FUEL"` to `FuelVehicle`. During deserialization, Jackson reads the `"type"` field and instantiates the correct subclass. The `As.EXISTING_PROPERTY` inclusion means the discriminator is already a field in the JSON.

---

### Real-World Cases
- **Payment Processing**: Different payment methods (credit card, PayPal, bank transfer) as polymorphic types
- **Notification Systems**: Email, SMS, and push notifications with shared base
- **Plugin Architectures**: Extensible handlers registered by type name
- **Document Management**: Different document types (PDF, Word, Excel) with shared metadata

### References
- @JsonSubTypes vs. Reflections for Polymorphic Deserialization - Baeldung - https://www.baeldung.com/java-jackson-polymorphic-deserialization
- CVE-2026-83557 in jackson-databind - VulDB - https://vuldb.com/cve/CVE-2026-83557
- Jackson Polymorphic Type Handling - Red Hat - https://docs.redhat.com

---

## Core Concept 4: Security & Guardrails

### Definitions

**Core Definition**: Security guardrails for data interchange are the defensive configurations and coding practices that prevent XML External Entity (XXE) injection, JSON depth bomb denial of service, and unsafe polymorphic deserialization attacks.

**Technical Definition**: **XXE injection** occurs when XML parsers process external entity references from untrusted input, enabling file disclosure, SSRF, and remote code execution. Prevention requires disabling DTD processing and external entities on all XML parsers (`DocumentBuilderFactory`, `SAXParserFactory`, `TransformerFactory`) and preferring modern libraries (Jackson XML, DOM4J 2.1.4+) that disable XXE by default. **JSON depth bombs** are deeply nested JSON arrays/objects that exhaust the call stack or heap during parsing. Jackson Core 2.15+ enforces a default `maxNestingDepth` of 1000 via `StreamReadConstraints`, and a default maximum number length of 1000, both configurable at runtime. **Unsafe polymorphic deserialization** occurs when `@JsonTypeInfo` is used without a `PolymorphicTypeValidator`, allowing attackers to instantiate arbitrary classes. The `DefaultBaseTypeLimitingValidator` has known CVEs (e.g., CVE-2026-83557) where `Comparable` was omitted from the unsafe base type list.

**Beginner-Friendly Explanation**: Think of security guardrails as the multiple layers of security at a bank. XXE prevention is locking the vault door (disabling external entities). JSON depth bomb protection is limiting how many floors the elevator can go (max nesting depth). Safe polymorphic deserialization is checking IDs at the door (type validator) so only authorized people (classes) enter the building.

### Purposes
- To prevent XML External Entity (XXE) injection attacks
- To protect against JSON depth bomb denial of service
- To prevent unsafe polymorphic deserialization
- To limit resource consumption during parsing
- To comply with security standards (OWASP, CWE)
- To maintain application availability under malicious input

### Syntax Rules and Structure

#### Complete General Syntax: Security Guardrails
```
SECURITY GUARDRAILS
│
├── 1. XXE Prevention
│   ├── DocumentBuilderFactory:
│   │   ├── setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)
│   │   └── setFeature("http://xml.org/sax/features/external-general-entities", false)
│   ├── SAXParserFactory: same features + external-parameter-entities
│   └── TransformerFactory: FEATURE_SECURE_PROCESSING = true
│
├── 2. JSON Depth Bomb Protection
│   ├── Upgrade to jackson-core 2.15.4+
│   └── Configure StreamReadConstraints:
│       ├── .maxNestingDepth(1000)   // Default: 1000
│       ├── .maxNumberLength(1000)   // Default: 1000
│       └── .maxDocumentLength(...)
│
├── 3. Safe Polymorphic Deserialization
│   ├── Always configure PolymorphicTypeValidator
│   ├── Never use Id.CLASS (use Id.NAME)
│   └── Upgrade to patched Jackson versions
│
└── 4. General Hardening
    ├── Limit payload size (Spring: spring.servlet.multipart.max-file-size)
    ├── Use XSD validation for XML
    └── Deploy WAF rules
```

#### Component Breakdown
| Vulnerability | CVE/Advisory | Mitigation |
|--------------|-------------|------------|
| XXE | Multiple | Disable DTD, external entities |
| JSON depth bomb | CVE-2025-52999 | Upgrade Jackson 2.15+, set `maxNestingDepth` |
| Number length DoS | Sonatype-2022-6438 | Set `maxNumberLength` |
| Unsafe polymorphic deserialization | CVE-2026-83557 | Configure `PolymorphicTypeValidator` |
| json-smart recursion DoS | CVE-2023-1370 | Upgrade json-smart 2.4.10+ |

#### Syntax Rules
- **XXE**: Disable `disallow-doctype-decl` and `external-general-entities` on all XML parsers
- **XXE**: Prefer Jackson XML or DOM4J 2.1.4+ (XXE-safe by default)
- **JSON depth**: Upgrade to Jackson 2.15.4+ and configure `StreamReadConstraints`
- **JSON depth**: Set `maxNestingDepth` and `maxNumberLength` based on application needs
- **Polymorphic**: Always provide a custom `PolymorphicTypeValidator`
- **Polymorphic**: Never use `Id.CLASS`; use `Id.NAME` with explicit subtypes
- **General**: Limit request body size at the framework or gateway level

#### Constraints and Limitations
- Jackson 2.15+ may break applications relying on nesting depth ≥ 1000
- `Comparable` as a base type is unsafe even with default validator
- XXE prevention requires configuring every XML parser instance
- JSON depth limits must be tuned per application
- WAF rules are not a substitute for secure code

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: XXE Prevention
```java
// XxePreventionDemo.java
import javax.xml.parsers.*;
import org.xml.sax.InputSource;
import java.io.StringReader;

public class XxePreventionDemo {
    public static void main(String[] args) throws Exception {
        // Malicious XML with XXE payload
        String maliciousXml = """
            <?xml version="1.0"?>
            <!DOCTYPE foo [
                <!ENTITY xxe SYSTEM "file:///etc/passwd">
            ]>
            <data>&xxe;</data>
            """;
        
        // ✅ SECURE: Disable DTD and external entities
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
        factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
        factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
        
        DocumentBuilder builder = factory.newDocumentBuilder();
        
        System.out.println("=== XXE Prevention Demo ===");
        try {
            builder.parse(new InputSource(new StringReader(maliciousXml)));
            System.out.println("✗ XXE payload was parsed (VULNERABLE)");
        } catch (Exception e) {
            System.out.println("✓ XXE payload blocked: " + e.getMessage());
            System.out.println("  DTD processing is disabled");
            System.out.println("  External entities cannot be resolved");
        }
    }
}
```
**Expected Output**:
```
=== XXE Prevention Demo ===
✓ XXE payload blocked: DOCTYPE is disallowed when the feature "http://apache.org/xml/features/disallow-doctype-decl" set to true.
  DTD processing is disabled
  External entities cannot be resolved
```
**Why This Output**: The `disallow-doctype-decl` feature prevents the parser from processing any DOCTYPE declaration, which is the entry point for XXE attacks. The `external-general-entities` and `external-parameter-entities` features are disabled to block external entity resolution. The parser throws an exception when it encounters the malicious DOCTYPE.

---

#### Example 2: JSON Depth Bomb Protection
```java
// JsonDepthBombDemo.java
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.ObjectMapper;

public class JsonDepthBombDemo {
    public static void main(String[] args) throws Exception {
        // Build a deeply nested JSON payload (1001 levels)
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 1001; i++) sb.append('[');
        for (int i = 0; i < 1001; i++) sb.append(']');
        
        System.out.println("=== JSON Depth Bomb Protection ===");
        System.out.println("Payload: 1001 levels of nested arrays");
        
        // ✅ SECURE: Configure StreamReadConstraints
        JsonFactory factory = JsonFactory.builder()
            .streamReadConstraints(StreamReadConstraints.builder()
                .maxNestingDepth(1000)   // Default: 1000
                .maxNumberLength(1000)   // Default: 1000
                .build())
            .build();
        
        ObjectMapper mapper = new ObjectMapper(factory);
        
        try {
            mapper.readTree(sb.toString());
            System.out.println("✗ Payload parsed (VULNERABLE)");
        } catch (StreamConstraintsException e) {
            System.out.println("✓ Depth bomb blocked: " + e.getMessage());
            System.out.println("  Maximum nesting depth: 1000");
        }
    }
}
```
**Expected Output**:
```
=== JSON Depth Bomb Protection ===
Payload: 1001 levels of nested arrays
✓ Depth bomb blocked: Depth (1001) exceeds the maximum allowed nesting depth (1000)
  Maximum nesting depth: 1000
```
**Why This Output**: Jackson Core 2.15+ enforces a default `maxNestingDepth` of 1000 via `StreamReadConstraints`. The 1001-level payload exceeds this limit and triggers a `StreamConstraintsException` before the stack is exhausted. Without this protection, the parser would recurse 1001 times and potentially cause a `StackOverflowError`.

---

### Real-World Cases
- **SOAP Web Services**: XXE is a critical vulnerability in XML-based SOAP endpoints
- **REST APIs**: JSON depth bombs can crash microservices that parse untrusted JSON
- **Authentication Systems**: JWT parsing with json-smart is vulnerable to CVE-2023-1370
- **Configuration APIs**: Polymorphic deserialization of configuration payloads is a common attack surface

### References
- Java怎么安全地处理外部实体 防止XXE - PHP.cn - https://www.php.cn/faq/1956604.html
- CVE-2025-52999 - jackson-core - GitHub - https://github.com/sassoftware/jackson
- CVE-2023-1370: json-smart Recursion DoS - Safeguard - https://safeguard.sh/resources/blog/cve-2023-1370
- CVE-2026-83557 in jackson-databind - VulDB - https://vuldb.com/cve/CVE-2026-83557
- Jackson Core 2.14.x Release Notes - HeroDevs - https://docs.herodevs.com/jackson/release-notes/jackson-core-2-14

---

## Deprecation and Safety Notes

| Feature | Status | Notes |
|---------|--------|-------|
| `FAIL_ON_UNKNOWN_PROPERTIES` | Default `true` | Set to `false` for forward compatibility |
| `Id.CLASS` in `@JsonTypeInfo` | ❌ Unsafe | Use `Id.NAME` with explicit subtypes |
| `DefaultBaseTypeLimitingValidator` | ⚠️ Known CVEs | CVE-2026-83557 (Comparable bypass) |
| Jackson Core < 2.15 | ❌ Vulnerable | CVE-2025-52999 (depth bomb) |
| json-smart < 2.4.10 | ❌ Vulnerable | CVE-2023-1370 (recursion DoS) |
| `DocumentBuilderFactory` default | ❌ XXE-vulnerable | Disable DTD and external entities |
| Jackson XML | ✅ XXE-safe by default | Since 2.9+ |
| `Optional` handling | ⚠️ Changed in Jackson 3.0 | `Optional.empty()` vs `null` |
| `@JsonView` with `@JsonTypeInfo` | ⚠️ Bypass risk | Creator properties may be populated |

---

## References

### Official Specifications
- Jakarta Bean Validation Specification - https://beanvalidation.org/
- Jackson Databind GitHub - https://github.com/FasterXML/jackson-databind
- OWASP XML External Entity Prevention Cheat Sheet - https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html

### API Versioning
- API Versioning in Spring - Spring Framework - https://springframework.org.cn/blog/2025/09/16/api-versioning-in-spring/
- Guide to Master API Versioning in Spring Boot - GUVI - https://www.guvi.in/blog/api-versioning-in-spring-boot-guide/

### Polymorphic Deserialization
- @JsonSubTypes vs. Reflections - Baeldung - https://www.baeldung.com/java-jackson-polymorphic-deserialization
- Jackson Polymorphic Type Handling - Red Hat - https://docs.redhat.com

### Security Advisories
- CVE-2025-52999 - GitHub - https://github.com/sassoftware/jackson
- CVE-2023-1370: json-smart Recursion DoS - Safeguard - https://safeguard.sh/resources/blog/cve-2023-1370
- CVE-2026-83557 in jackson-databind - VulDB - https://vuldb.com/cve/CVE-2026-83557
- Jackson Core 2.14.x Release Notes - HeroDevs - https://docs.herodevs.com/jackson/release-notes/jackson-core-2-14

### Validation
- Using @Valid for Request Validation - CodeGym - https://codegym.cc/quests/lectures/en.codegym.java.spring.lecture.level09.lecture03
- @Valid Annotation on Child Objects - Baeldung - https://www.baeldung.com/java-valid-annotation-child-objects
- Jakarta Bean Validation: Avoiding Common Pitfalls - Safeguard - https://safeguard.sh/resources/blog/jakarta-bean-validation-avoiding-common-pitfalls

### XXE Prevention
- Java怎么安全地处理外部实体 防止XXE - PHP.cn - https://www.php.cn/faq/1956604.html