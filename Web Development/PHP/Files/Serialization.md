# PHP Serialization — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP Serialization is the process of converting a PHP value — scalar, array, or object — into a storable byte-stream string representation that preserves both the value and its type structure. Deserialization is the reverse process, reconstructing the original PHP value from the serialized string.

**Technical Definition**  
PHP serialization is implemented by the `serialize()` function, which produces a format-specific string encoding each PHP type with a type marker followed by its data. `unserialize()` parses this string and reconstructs the value, invoking object `__wakeup()` or `__unserialize()` magic methods during reconstruction. The format is PHP-specific and not interchangeable with other serialization formats (JSON, XML, MessagePack). PHP 7.4 introduced `__serialize()` and `__unserialize()` as the modern replacement for the deprecated `Serializable` interface. The `unserialize()` function supports an `allowed_classes` option (PHP 7.0+) that restricts which classes may be instantiated during deserialization, providing defense against PHP Object Injection attacks.

**Beginner-Friendly Explanation**  
Imagine you have a complex LEGO model — a car, a house, a robot. To store it or send it to a friend, you can’t just hand over the assembled model; you need to break it down into a flat list of bricks with instructions. Serialization does exactly that: it takes a PHP value (a string, an array, an object) and turns it into a string you can save to a file, put in a database, or send over a network. Later, you can use that string to rebuild the original value exactly as it was. The catch is that if you rebuild a serialized string from an untrusted source, an attacker might have hidden a dangerous “brick” that, when assembled, runs malicious code. This is why deserialization of untrusted data is a well-known security risk.

---

### Key Characteristics

- `serialize()` handles all PHP types except `resource` and some internal objects.
- The serialized format is PHP-specific and not human-readable.
- Object serialization invokes `__sleep()` (legacy) or `__serialize()` (modern, PHP 7.4+) before serialization.
- Deserialization invokes `__wakeup()` (legacy) or `__unserialize()` (modern, PHP 7.4+) after reconstruction.
- PHP Object Injection is the primary security risk: crafted serialized data can instantiate arbitrary classes and trigger gadget chains.
- `unserialize()` must never be called on user-controlled input without `allowed_classes` restrictions.
- The `Serializable` interface is deprecated as of PHP 8.1.0; use `__serialize()` and `__unserialize()` instead.
- `serialize()` is generally faster than `json_encode()` for PHP-native data; JSON is more portable and safer.
- `__PHP_Incomplete_Class` is used when an object’s class is not available during deserialization.
- Session data is serialized automatically by PHP using the configured `session.serialize_handler`.
- PHP 8.4.0 introduced stricter handling of non-array `allowed_classes` values.

---

### Prerequisites

- Basic PHP syntax: variables, arrays, objects, magic methods.
- Object-oriented PHP: classes, properties, methods.
- Basic HTTP and session concepts.
- Understanding of filesystem and database storage.
- Awareness of web application security (OWASP Top 10).

---

### Related Programming Areas

- PHP Object Injection and deserialization attacks.
- Session management (`session_start()`, session serialization).
- Data persistence (files, databases, caches).
- API data exchange (JSON, XML).
- PHP security hardening (OWASP, CWE-502).
- Message queuing and distributed systems.

---

## Core Concepts / Features

## 1. The Serialization Format — Structure and Type Markers

### Definitions

**Core Definition**  
The PHP serialization format is a string-based encoding that represents PHP values using type markers, length prefixes, and delimited data.

**Technical Definition**  
Each PHP type is encoded with a specific marker. Scalars use single-character markers (`i` for integer, `d` for double, `s` for string, `b` for boolean, `N` for null). Arrays use `a:size:{key;value;...}`. Objects use `O:length:"ClassName":property_count:{...}`. Custom serialization via `Serializable` uses `C:length:"ClassName":data_length:{...}`. PHP 7.4+ `__serialize()` produces an array that is then serialized, still using the `O:` marker but with different internal structure. The format is not self-describing across PHP versions and is not portable to other languages.

**Beginner-Friendly Explanation**  
The serialized string is like a recipe written in a secret code. Each ingredient is marked with a letter: `i` means integer, `s` means string, `a` means array, `O` means object. The recipe includes the exact size of each item so the decoder knows where one ends and the next begins. This makes the format compact but unreadable to humans and incompatible with other programming languages.

### Purposes

- To store PHP values in a compact, type-preserving format.
- To pass complex data structures between PHP processes.
- To persist objects and arrays in files or databases.
- To enable session data storage.
- To cache computed results.
- To implement deep cloning via `serialize()`/`unserialize()`.
- To test object state round-tripping.

### Syntax Rules and Structure

**General Syntax: Serialized Type Markers**
```
N;                          // null
b:0;  b:1;                  // boolean false / true
i:42;                       // integer
d:3.14;                     // double (float)
s:5:"hello";                // string (length:value)
a:2:{i:0;s:3:"foo";i:1;s:3:"bar";}  // array
O:8:"stdClass":1:{s:4:"name";s:5:"Alice";}  // object
C:3:"Foo":10:{...}          // Serializable interface
```

**Component Breakdown**

- `N;` — null; no data follows.
- `b:0;` / `b:1;` — boolean false / true.
- `i:42;` — integer 42.
- `d:3.14;` — double 3.14.
- `s:5:"hello";` — string of length 5, content `hello`.
- `a:2:{...}` — array with 2 elements; each element is a key-value pair.
- `O:8:"stdClass":1:{...}` — object of class `stdClass` with 1 property.
- `C:3:"Foo":10:{...}` — object implementing `Serializable` with 10 bytes of custom data.

**General Syntax: `serialize()`**
```php
string serialize(mixed $value)
```

**Component Breakdown**

- `$value` — the value to serialize.
- Returns a string containing the byte-stream representation.
- Handles all types except `resource` and some internal objects.

**General Syntax: `unserialize()`**
```php
mixed unserialize(string $data, array $options = [])
```

**Component Breakdown**

- `$data` — the serialized string.
- `$options` — associative array with `allowed_classes` and `max_depth`.
- `allowed_classes` — `true` (default, accept all), `false` (accept none), or an array of class names.
- `max_depth` — maximum nesting depth (default 4096; `0` disables the limit).
- Returns the reconstructed value, or `false` on failure.

**Syntax Rules**

- String lengths in `s:` are byte counts, not character counts.
- Array keys can be integers or strings; the format distinguishes them.
- Object property names include visibility markers (`\0ClassName\0property` for private).
- References are preserved via `R:` and `r:` markers.
- The format is not canonical; different serializations can produce the same value.
- `serialize()` cannot handle `resource` types or closures.
- `__PHP_Incomplete_Class` is used when the class is not defined.

**Constraints and Limitations**

- The format is PHP-specific and not portable.
- Serialized strings are not human-readable.
- The format is not self-validating; malformed strings can cause errors.
- `unserialize()` returns `false` for malformed data.
- Large or deeply nested structures can cause memory issues.
- The format does not include a version number; changes across PHP versions can cause incompatibilities.
- `serialize()` on objects with circular references can cause issues without proper `__sleep()`/`__serialize()`.

### Annotated Code Examples and Expected Outputs

#### Example 1: Serializing Scalars, Arrays, and Objects

**File: `serialize_demo.php`**
```php
<?php
// Scalar.
echo serialize(42) . "\n";           // i:42;
echo serialize(3.14) . "\n";         // d:3.14;
echo serialize("hello") . "\n";      // s:5:"hello";
echo serialize(true) . "\n";         // b:1;
echo serialize(null) . "\n";         // N;

// Array.
$array = ['name' => 'Alice', 'age' => 30];
echo serialize($array) . "\n";
// a:2:{s:4:"name";s:5:"Alice";s:3:"age";i:30;}

// Object.
$obj = new stdClass();
$obj->name = 'Alice';
$obj->age = 30;
echo serialize($obj) . "\n";
// O:8:"stdClass":2:{s:4:"name";s:5:"Alice";s:3:"age";i:30;}
?>
```

**Expected Output**
```
i:42;
d:3.14;
s:5:"hello";
b:1;
N;
a:2:{s:4:"name";s:5:"Alice";s:3:"age";i:30;}
O:8:"stdClass":2:{s:4:"name";s:5:"Alice";s:3:"age";i:30;}
```

**Why This Output Occurs**

- Each type is encoded with its marker and data.
- Strings include their byte length.
- Arrays and objects include their size and element/property pairs.

#### Example 2: Round-Tripping a Value

**File: `roundtrip.php`**
```php
<?php
$original = [
    'users' => [
        ['id' => 1, 'name' => 'Alice'],
        ['id' => 2, 'name' => 'Bob']
    ],
    'count' => 2
];

$serialized = serialize($original);
echo 'Serialized: ' . $serialized . "\n\n";

$restored = unserialize($serialized);
var_dump($restored === $original); // true
?>
```

**Expected Output**
```
Serialized: a:2:{s:5:"users";a:2:{i:0;a:2:{s:2:"id";i:1;s:4:"name";s:5:"Alice";}i:1;a:2:{s:2:"id";i:2;s:4:"name";s:3:"Bob";}}s:5:"count";i:2;}

bool(true)
```

**Why This Output Occurs**

- `serialize()` produces a string representation of the entire structure.
- `unserialize()` reconstructs an identical value.
- The `===` comparison confirms type and value equality.

### Real-World Cases

- **Session storage:** PHP serializes session data automatically.
- **Cache systems:** Store serialized arrays or objects in Redis/Memcached.
- **Database storage:** Store serialized preferences or settings.
- **Deep cloning:** Use `unserialize(serialize($obj))` to clone an object.
- **Message queues:** Serialize payloads for asynchronous processing.

---

## 2. Object Serialization — Magic Methods and Interfaces

### Definitions

**Core Definition**  
Object serialization in PHP is controlled by magic methods (`__sleep()`, `__wakeup()`, `__serialize()`, `__unserialize()`) and the `Serializable` interface, allowing classes to customize what data is serialized and how it is reconstructed.

**Technical Definition**  
Before serialization, PHP checks for `__serialize()` (PHP 7.4+). If present, it calls the method and serializes the returned array. Otherwise, it checks for `__sleep()`, which returns an array of property names to serialize. On deserialization, `__unserialize()` (PHP 7.4+) is called with the serialized array, or `__wakeup()` is called after reconstruction. The `Serializable` interface (deprecated in PHP 8.1.0) uses `serialize()` and `unserialize()` methods instead.

**Beginner-Friendly Explanation**  
When you serialize an object, PHP needs to know which properties to save and whether to do anything special before saving (like closing a database connection). Magic methods let you control this. `__sleep()` says “here are the properties to save.” `__wakeup()` says “now that I’m rebuilt, here’s how to reinitialize.” The modern `__serialize()` and `__unserialize()` methods are more flexible and are the recommended approach.

### Purposes

- To exclude sensitive or non-serializable properties from serialization.
- To reinitialize resources (database connections, file handles) after deserialization.
- To control the serialized representation of complex objects.
- To implement deep cloning with custom logic.
- To support backwards compatibility with older serialized data.
- To prevent serialization of objects that should not be persisted.
- To validate deserialized data before use.

### Syntax Rules and Structure

**General Syntax: `__sleep()` and `__wakeup()` (Legacy)**
```php
public function __sleep(): array {
    return ['property1', 'property2'];
}

public function __wakeup(): void {
    // Reinitialize resources.
}
```

**Component Breakdown**

- `__sleep()` — returns an array of property names to serialize.
- `__wakeup()` — called after reconstruction; reinitializes resources.
- PHP 8.0+ issues a warning if `__sleep()` does not return an array.

**General Syntax: `__serialize()` and `__unserialize()` (Modern, PHP 7.4+)**
```php
public function __serialize(): array {
    return ['property1' => $this->property1, 'property2' => $this->property2];
}

public function __unserialize(array $data): void {
    $this->property1 = $data['property1'];
    $this->property2 = $data['property2'];
}
```

**Component Breakdown**

- `__serialize()` — returns an array of data to serialize.
- `__unserialize()` — receives the array and restores the object.
- Preferred over `__sleep()`/`__wakeup()`.

**General Syntax: `Serializable` Interface (Deprecated in PHP 8.1.0)**
```php
class Foo implements Serializable {
    public function serialize(): string {
        return serialize($this->data);
    }
    public function unserialize(string $data): void {
        $this->data = unserialize($data);
    }
}
```

**Component Breakdown**

- `serialize()` — returns a string representation.
- `unserialize()` — reconstructs from the string.
- Deprecated in PHP 8.1.0; use `__serialize()`/`__unserialize()`.

**Magic Method Call Order**

| Method | When Called | PHP Version |
|---|---|---|
| `__serialize()` | Before serialization | 7.4+ |
| `__sleep()` | Before serialization (if no `__serialize()`) | 4.0.7+ |
| `__unserialize()` | After deserialization | 7.4+ |
| `__wakeup()` | After deserialization (if no `__unserialize()`) | 4.0+ |
| `__destruct()` | On object destruction | 5.0+ |
| `__toString()` | When object is used as string | 5.2+ |

**Syntax Rules**

- `__serialize()` takes precedence over `__sleep()`.
- `__unserialize()` takes precedence over `__wakeup()`.
- If neither is defined, all properties are serialized.
- `__sleep()` must return an array of property names.
- `__wakeup()` must not return a value.
- `Serializable` interface methods are called instead of magic methods.
- PHP 8.1.0 deprecates `Serializable` without `__serialize()`/`__unserialize()`.

**Constraints and Limitations**

- `__sleep()` cannot return private properties of parent classes.
- `__wakeup()` cannot restore resources that are not serializable.
- `Serializable` is deprecated; migrate to `__serialize()`/`__unserialize()`.
- Closures and resources cannot be serialized.
- Circular references require careful handling.
- The serialized format of `Serializable` objects uses `C:` notation, which differs from `O:` notation.

### Annotated Code Examples and Expected Outputs

#### Example 1: `__sleep()` and `__wakeup()`

**File: `sleep_wakeup.php`**
```php
<?php
class User {
    public $name;
    public $password; // Should not be serialized.
    private $db;      // Not serializable.

    public function __construct($name, $password) {
        $this->name = $name;
        $this->password = $password;
        $this->db = new PDO('sqlite::memory:');
    }

    public function __sleep(): array {
        return ['name']; // Only serialize the name.
    }

    public function __wakeup(): void {
        // Reinitialize the database connection.
        $this->db = new PDO('sqlite::memory:');
    }
}

$user = new User('Alice', 'secret');
$serialized = serialize($user);
echo $serialized . "\n";

$restored = unserialize($serialized);
echo 'Name: ' . $restored->name . "\n";
echo 'Password: ' . ($restored->password ?? 'not set') . "\n";
?>
```

**Expected Output**
```
O:4:"User":1:{s:4:"name";s:5:"Alice";}
Name: Alice
Password: not set
```

**Why This Output Occurs**

- `__sleep()` returns only `['name']`, so `password` and `db` are excluded.
- `__wakeup()` reinitializes the database connection.
- `password` is not serialized, so it is `null` after restoration.

#### Example 2: `__serialize()` and `__unserialize()`

**File: `serialize_methods.php`**
```php
<?php
class Config {
    public array $settings;
    private string $secretKey;

    public function __construct(array $settings, string $secretKey) {
        $this->settings = $settings;
        $this->secretKey = $secretKey;
    }

    public function __serialize(): array {
        // Do not serialize the secret key.
        return ['settings' => $this->settings];
    }

    public function __unserialize(array $data): void {
        $this->settings = $data['settings'];
        $this->secretKey = ''; // Reset; must be reloaded.
    }
}

$config = new Config(['theme' => 'dark'], 'super-secret');
$serialized = serialize($config);
echo $serialized . "\n";

$restored = unserialize($serialized);
var_dump($restored->settings);
?>
```

**Expected Output**
```
O:6:"Config":1:{s:8:"settings";a:1:{s:5:"theme";s:4:"dark";}}
array(1) { ["theme"]=> string(4) "dark" }
```

**Why This Output Occurs**

- `__serialize()` returns only the `settings` array.
- `__unserialize()` restores `settings` and resets `secretKey`.
- The `secretKey` is never serialized.

### Real-World Cases

- **Authentication objects:** Exclude password hashes from serialization.
- **Database connections:** Reinitialize connections in `__wakeup()`.
- **Configuration objects:** Exclude secrets from serialized cache.
- **DTOs:** Use `__serialize()` for clean, explicit serialization.
- **Legacy code:** Migrate from `Serializable` to `__serialize()`/`__unserialize()`.

---

## 3. Security — PHP Object Injection and Deserialization Attacks

### Definitions

**Core Definition**  
PHP Object Injection is a vulnerability that occurs when untrusted data is passed to `unserialize()`, allowing an attacker to instantiate arbitrary classes and potentially execute arbitrary code via gadget chains.

**Technical Definition**  
When `unserialize()` processes a serialized object, PHP instantiates the class and calls its magic methods (`__wakeup()`, `__destruct()`, `__toString()`, `__unserialize()`). An attacker can craft a serialized payload that instantiates a class with malicious property values, triggering dangerous behavior in those magic methods. By chaining multiple classes (a “gadget chain”), an attacker can escalate to Remote Code Execution (RCE). This is classified as CWE-502: Deserialization of Untrusted Data. The `allowed_classes` option (PHP 7.0+) restricts which classes may be instantiated, but does not fully eliminate the risk if allowed classes have dangerous magic methods.

**Beginner-Friendly Explanation**  
Imagine you receive a package that claims to contain a harmless book. When you open it, a spring-loaded boxing glove pops out. In PHP, `unserialize()` is like opening that package. An attacker can send a serialized string that looks harmless but contains instructions to create objects that, when “opened” (deserialized), run malicious code. The `allowed_classes` option is like a security scanner that only lets through books from approved publishers — but even approved books can contain surprises if their authors wrote dangerous code in `__wakeup()`.

### Purposes

- To understand the attack surface of `unserialize()`.
- To identify and mitigate PHP Object Injection vulnerabilities.
- To implement safe deserialization practices.
- To audit code for insecure `unserialize()` calls.
- To comply with OWASP and CWE security standards.
- To protect session data and cached objects.
- To prevent RCE via gadget chains.

### Syntax Rules and Structure

**General Syntax: Vulnerable Code**
```php
// NEVER DO THIS.
$data = unserialize($_COOKIE['user_data']);
```

**Component Breakdown**

- `$_COOKIE['user_data']` — user-controlled input.
- `unserialize()` — instantiates arbitrary classes.
- An attacker can inject a serialized payload to trigger gadget chains.

**General Syntax: Safe Deserialization with `allowed_classes`**
```php
$data = unserialize($input, [
    'allowed_classes' => false  // No classes allowed.
]);

// Or with an allowlist:
$data = unserialize($input, [
    'allowed_classes' => [MyModel::class, DateTime::class]
]);
```

**Component Breakdown**

- `'allowed_classes' => false` — no classes instantiated; objects become `__PHP_Incomplete_Class`.
- `'allowed_classes' => [...]` — only listed classes instantiated.
- `max_depth` — limits nesting to prevent stack overflow.

**General Syntax: HMAC Verification**
```php
$serialized = $_COOKIE['data'];
$hmac = $_COOKIE['hmac'] ?? '';
$expected = hash_hmac('sha256', $serialized, $secretKey);

if (hash_equals($expected, $hmac)) {
    $data = unserialize($serialized, ['allowed_classes' => false]);
}
```

**Component Breakdown**

- `hash_hmac()` — generates an HMAC of the serialized data.
- `hash_equals()` — timing-safe comparison.
- Only deserialize if the HMAC matches.

**Common Gadget Chain Classes**

| Class | Magic Method | Potential Impact |
|---|---|---|
| `__destruct()` | Destructor | File deletion, code execution |
| `__wakeup()` | Wakeup | Resource initialization, code execution |
| `__toString()` | String conversion | SQL injection, file read |
| `__call()` | Method call | Arbitrary method invocation |
| `__get()` | Property access | Information disclosure |
| `__invoke()` | Function call | Code execution |

**Syntax Rules**

- Never call `unserialize()` on user-controlled input.
- Use `allowed_classes => false` if deserialization is unavoidable.
- Use an allowlist of specific classes if objects are needed.
- Use `hash_hmac()` to verify the integrity of serialized data.
- Prefer JSON (`json_decode()`) for data exchange.
- Use `max_depth` to prevent stack overflow.
- Audit all `unserialize()` calls in your codebase.
- Monitor for `__PHP_Incomplete_Class` in logs.

**Constraints and Limitations**

- `allowed_classes` only applies to the outermost `unserialize()` call; nested calls do not enforce it.
- `allowed_classes` does not prevent gadget chains if allowed classes have dangerous magic methods.
- HMAC verification requires secure key management.
- On PHP < 7.0, `allowed_classes` does not exist.
- `unserialize()` can still cause denial of service via large payloads.
- `__PHP_Incomplete_Class` objects can still trigger `__destruct()` in some cases.

### Annotated Code Examples and Expected Outputs

#### Example 1: Vulnerable Code

**File: `vulnerable.php`**
```php
<?php
// DANGEROUS: user-controlled data passed to unserialize().
class Logger {
    public $logFile;
    public function __destruct() {
        // Deletes the file on destruction!
        if ($this->logFile) {
            unlink($this->logFile);
        }
    }
}

$data = $_GET['data'] ?? '';
$obj = unserialize($data);
echo 'Deserialized.';
?>
```

**Attack Payload**
```
O:6:"Logger":1:{s:7:"logFile";s:11:"/etc/passwd";}
```

**Expected Output (if attack succeeds)**
```
Deserialized.
```
The file `/etc/passwd` is deleted when the object is destroyed.

**Why This Output Occurs**

- The attacker crafts a serialized `Logger` object with `logFile` pointing to a sensitive file.
- `unserialize()` instantiates the object.
- When the script ends, `__destruct()` is called, deleting the file.

#### Example 2: Safe Deserialization

**File: `safe.php`**
```php
<?php
class SafeData {
    public string $name;
    public function __construct(string $name = '') {
        $this->name = $name;
    }
}

$data = $_GET['data'] ?? '';

// Only allow SafeData objects.
$obj = unserialize($data, [
    'allowed_classes' => [SafeData::class],
    'max_depth' => 10
]);

if ($obj instanceof SafeData) {
    echo 'Name: ' . htmlspecialchars($obj->name, ENT_QUOTES, 'UTF-8');
} else {
    echo 'Invalid data.';
}
?>
```

**Expected Output (valid payload)**
```
Name: Alice
```
**Expected Output (malicious payload)**
```
Invalid data.
```

**Why This Output Occurs**

- `allowed_classes` restricts instantiation to `SafeData`.
- A malicious `Logger` object becomes `__PHP_Incomplete_Class`, which is not an instance of `SafeData`.
- The script rejects it.

### Real-World Cases

- **CVE-2025-48951:** Auth0-PHP SDK insecure deserialization of cookie data, allowing crafted cookies to trigger malicious serialized data.
- **CVE-2026-33942:** Saloon `AccessTokenAuthenticator` insecure deserialization, enabling RCE via gadget chains.
- **CVE-2025-34292:** BeWelcome/Rox PHP Object Injection RCE from untrusted data.
- **CVE-2025-25034:** SugarCRM deserialization of untrusted data.
- **Session file injection:** Crafted `X-Forwarded-For` header injecting serialized payload into session files.

---

## 4. Modern Alternatives — `__serialize()`/`__unserialize()` and JSON

### Definitions

**Core Definition**  
`__serialize()` and `__unserialize()` are modern magic methods (PHP 7.4+) that provide a clean, flexible replacement for the deprecated `Serializable` interface. JSON (`json_encode()`/`json_decode()`) is a language-agnostic alternative for data interchange that does not carry the same deserialization risks.

**Technical Definition**  
`__serialize()` returns an array of data to be serialized; `__unserialize(array $data)` reconstructs the object from that array. These methods take precedence over `__sleep()` and `__wakeup()`. JSON encoding produces a string that is human-readable, cross-platform, and free of the object-injection risks inherent in PHP’s native serialization. However, JSON does not preserve object types or class identity; it must be reconstructed manually.

**Beginner-Friendly Explanation**  
The old way to customize serialization was `Serializable`, which is now deprecated. The new way is `__serialize()` and `__unserialize()`, which are simpler and safer. If you only need to store simple data (arrays, strings, numbers), JSON is an even better choice: it is readable by other languages, it is safer because it cannot instantiate arbitrary PHP classes, and it is supported everywhere.

### Purposes

- To replace the deprecated `Serializable` interface.
- To provide explicit control over serialized data.
- To support versioned serialization formats.
- To store data in a portable, language-agnostic format (JSON).
- To eliminate object-injection risks when deserializing.
- To improve interoperability with other systems.
- To simplify testing and debugging.

### Syntax Rules and Structure

**General Syntax: `__serialize()` and `__unserialize()`**
```php
class MyClass {
    public function __serialize(): array {
        return ['prop1' => $this->prop1, 'prop2' => $this->prop2];
    }

    public function __unserialize(array $data): void {
        $this->prop1 = $data['prop1'];
        $this->prop2 = $data['prop2'];
    }
}
```

**Component Breakdown**

- `__serialize()` — returns an array representing the object’s state.
- `__unserialize()` — restores the object from the array.
- Both are called automatically by `serialize()`/`unserialize()`.

**General Syntax: JSON Encoding**
```php
$json = json_encode($data, JSON_THROW_ON_ERROR);
$data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
```

**Component Breakdown**

- `json_encode()` — converts PHP values to a JSON string.
- `json_decode()` — converts JSON back to PHP values.
- `JSON_THROW_ON_ERROR` — throws `JsonException` on error instead of returning `false`.
- `true` — decode objects as associative arrays.

**General Syntax: `JsonSerializable` Interface**
```php
class User implements JsonSerializable {
    public function jsonSerialize(): mixed {
        return ['name' => $this->name, 'email' => $this->email];
    }
}
```

**Component Breakdown**

- `JsonSerializable` — controls JSON representation.
- `jsonSerialize()` — returns the data to encode.

**Feature Comparison: `serialize()` vs `json_encode()`**

| Feature | `serialize()` | `json_encode()` |
|---|---|---|
| Format | PHP-specific | JSON (RFC 8259) |
| Human-readable | No | Yes |
| Cross-language | No | Yes |
| Preserves object types | Yes | No |
| Preserves class identity | Yes | No |
| Security risk | High (object injection) | Low |
| Performance | Faster for PHP-native data | Slower but portable |
| Session storage | Default handler | Not used by default |
| Deep nesting | Supports references | Limited depth (512 default) |
| Binary data | Supports binary strings | Requires base64 encoding |

**Syntax Rules**

- `__serialize()` takes precedence over `__sleep()`.
- `__unserialize()` takes precedence over `__wakeup()`.
- `Serializable` is deprecated; migrate to `__serialize()`/`__unserialize()`.
- JSON does not preserve object types; reconstruct manually.
- Use `JSON_THROW_ON_ERROR` for error handling.
- Implement `JsonSerializable` for custom JSON output.
- JSON is safer for data crossing trust boundaries.

**Constraints and Limitations**

- `__serialize()`/`__unserialize()` require PHP 7.4+.
- JSON cannot represent resources, closures, or binary data directly.
- JSON does not preserve object class identity.
- JSON has a default depth limit of 512.
- `serialize()` is faster for PHP-native data; JSON is more portable.
- `json_decode()` with `true` returns arrays, not objects.
- JSON numbers may lose precision for very large integers.

### Annotated Code Examples and Expected Outputs

#### Example 1: Modern `__serialize()`/`__unserialize()`

**File: `modern_serialize.php`**
```php
<?php
class Product {
    public string $name;
    public float $price;
    private string $internalCode;

    public function __construct(string $name, float $price, string $code) {
        $this->name = $name;
        $this->price = $price;
        $this->internalCode = $code;
    }

    public function __serialize(): array {
        return ['name' => $this->name, 'price' => $this->price];
    }

    public function __unserialize(array $data): void {
        $this->name = $data['name'];
        $this->price = $data['price'];
        $this->internalCode = 'REGENERATED';
    }
}

$product = new Product('Widget', 19.99, 'XYZ-123');
$serialized = serialize($product);
echo $serialized . "\n";

$restored = unserialize($serialized);
var_dump($restored);
?>
```

**Expected Output**
```
O:7:"Product":2:{s:4:"name";s:6:"Widget";s:5:"price";d:19.99;}
object(Product)#2 (3) {
  ["name"]=> string(6) "Widget"
  ["price"]=> float(19.99)
  ["internalCode":"Product":private]=> string(11) "REGENERATED"
}
```

**Why This Output Occurs**

- `__serialize()` returns only `name` and `price`.
- `__unserialize()` restores those and regenerates `internalCode`.
- The serialized string does not contain `internalCode`.

#### Example 2: JSON Round-Trip

**File: `json_roundtrip.php`**
```php
<?php
$data = [
    'users' => [
        ['id' => 1, 'name' => 'Alice'],
        ['id' => 2, 'name' => 'Bob']
    ],
    'count' => 2
];

$json = json_encode($data, JSON_PRETTY_PRINT);
echo $json . "\n\n";

$restored = json_decode($json, true);
var_dump($restored === $data);
?>
```

**Expected Output**
```
{
    "users": [
        {
            "id": 1,
            "name": "Alice"
        },
        {
            "id": 2,
            "name": "Bob"
        }
    ],
    "count": 2
}

bool(true)
```

**Why This Output Occurs**

- `json_encode()` produces a human-readable JSON string.
- `json_decode($json, true)` returns an associative array.
- The round-trip preserves the data exactly.

### Real-World Cases

- **API communication:** JSON is the standard for REST APIs.
- **Configuration files:** Store settings as JSON.
- **Cache systems:** Use JSON for cross-language cache compatibility.
- **Session storage:** Use JSON for simple session data (with custom handlers).
- **Data export:** JSON is readable and portable.

---

## 5. Session Serialization

### Definitions

**Core Definition**  
Session serialization is the process by which PHP converts the `$_SESSION` superglobal array into a storable format when the request ends, and reconstructs it when the session is resumed.

**Technical Definition**  
PHP’s session extension uses a `session.serialize_handler` directive to determine the serialization format. The default is `php` (the `serialize()` format). Alternative handlers include `php_binary`, `php_serialize`, `wddx`, and `igbinary`. Session data is stored server-side (files, Redis, etc.) and is subject to the same object-injection risks if an attacker can write to the session storage. Historically, session serialization has been the source of several CVEs, including CVE-2010-3065 and CVE-2007-1380.

**Beginner-Friendly Explanation**  
When a user visits your site, PHP stores their session data (like their user ID or cart contents) in a file or database. To do this, it converts the `$_SESSION` array into a string using a serialization handler. If an attacker can manipulate this serialized data — for example, by injecting a crafted session file — they might be able to trigger object injection. This is why session storage must be secured and why the `allowed_classes` option should be used where possible.

### Purposes

- To persist session data across requests.
- To store user-specific state server-side.
- To support shopping carts, authentication, and multi-step forms.
- To choose an efficient serialization handler for performance.
- To secure session data against tampering.
- To migrate session storage between backends.
- To audit session serialization for security risks.

### Syntax Rules and Structure

**General Syntax: `php.ini` Session Serialization Configuration**
```ini
session.serialize_handler = php
```

**Component Breakdown**

- `php` — default; uses the `serialize()` format.
- `php_binary` — binary format; more compact but less portable.
- `php_serialize` — uses `serialize()` directly; available since PHP 5.5.4.
- `igbinary` — requires the igbinary extension; faster and more compact.
- `wddx` — deprecated; XML-based.

**General Syntax: Changing the Handler at Runtime**
```php
ini_set('session.serialize_handler', 'php_serialize');
session_start();
```

**Component Breakdown**

- `ini_set()` — changes the handler for the current request.
- Must be called before `session_start()`.
- Changing the handler mid-session can corrupt data.

**General Syntax: Custom Session Handler with JSON**
```php
class JsonSessionHandler implements SessionHandlerInterface {
    public function read($id): string {
        $data = file_get_contents($this->path . '/' . $id);
        return $data ? json_decode($data, true) : [];
    }
    public function write($id, $data): bool {
        return file_put_contents($this->path . '/' . $id, json_encode($data)) !== false;
    }
    // ... other methods
}
```

**Component Breakdown**

- `SessionHandlerInterface` — requires `open`, `close`, `read`, `write`, `destroy`, `gc`.
- `json_encode()`/`json_decode()` — replace PHP serialization.
- Avoids object-injection risks.

**Session Serialization Security Issues**

| CVE | Description |
|---|---|
| CVE-2010-3065 | Default session serializer mishandles `PS_UNDEF_MARKER`, allowing variable injection |
| CVE-2007-1380 | `php_binary` handler allows memory disclosure via large length values |
| CVE-2015-XXXX | Switching serialization handlers mid-session causes data injection |

**Syntax Rules**

- Choose a serialization handler before starting the session.
- Do not switch handlers mid-session.
- Use `php_serialize` for better security than the default `php` handler.
- Use `igbinary` for performance if available.
- Secure session storage (files, Redis) with appropriate permissions.
- Use HTTPS to protect session cookies.
- Regenerate session IDs after login.
- Store sessions outside the web root.

**Constraints and Limitations**

- The default `php` handler is not the most secure.
- `php_binary` is not portable and has had security issues.
- `igbinary` requires a PECL extension.
- `wddx` is deprecated.
- Session data is not encrypted by default.
- Session files can be read by other users if permissions are incorrect.

### Annotated Code Examples and Expected Outputs

#### Example 1: Session Serialization with `php_serialize`

**File: `session_config.php`**
```php
<?php
// Use the safer php_serialize handler.
ini_set('session.serialize_handler', 'php_serialize');

session_start();

$_SESSION['user'] = ['id' => 1, 'name' => 'Alice'];
echo 'Session serialization handler: ' . ini_get('session.serialize_handler') . '<br>';
echo 'Session ID: ' . session_id();
?>
```

**Expected Output**
```
Session serialization handler: php_serialize
Session ID: abc123def456
```

**Why This Output Occurs**

- `ini_set()` changes the handler before `session_start()`.
- `php_serialize` uses the standard `serialize()` format more directly.
- Session data is stored with the configured handler.

#### Example 2: Custom JSON Session Handler

**File: `json_session.php`**
```php
<?php
class JsonSessionHandler implements SessionHandlerInterface {
    private string $path;

    public function __construct(string $path) {
        $this->path = $path;
        if (!is_dir($path)) mkdir($path, 0700, true);
    }

    public function open($savePath, $sessionName): bool { return true; }
    public function close(): bool { return true; }

    public function read($id): string {
        $file = $this->path . '/' . $id;
        if (!file_exists($file)) return '';
        $json = file_get_contents($file);
        return json_decode($json, true) ?: [];
    }

    public function write($id, $data): bool {
        return file_put_contents($this->path . '/' . $id, json_encode($data)) !== false;
    }

    public function destroy($id): bool {
        return unlink($this->path . '/' . $id);
    }

    public function gc($max_lifetime): int|false {
        foreach (glob($this->path . '/*') as $file) {
            if (filemtime($file) + $max_lifetime < time()) unlink($file);
        }
        return true;
    }
}

$handler = new JsonSessionHandler(__DIR__ . '/sessions');
session_set_save_handler($handler, true);
session_start();

$_SESSION['cart'] = ['item1', 'item2'];
echo 'Session stored as JSON.';
?>
```

**Expected Output**
```
Session stored as JSON.
```

**Why This Output Occurs**

- The custom handler stores session data as JSON.
- JSON is safer than PHP serialization for session data.
- The session file contains readable JSON.

### Real-World Cases

- **High-security applications:** Use `php_serialize` or a custom JSON handler.
- **Performance-critical apps:** Use `igbinary` for faster serialization.
- **Multi-server deployments:** Use Redis or Memcached with a shared handler.
- **Compliance:** Use JSON for audit-friendly session storage.
- **Legacy systems:** Audit for `php_binary` handler vulnerabilities.

---

## Security Considerations

### PHP Object Injection

- **Attack:** Crafted serialized data instantiates arbitrary classes, triggering dangerous magic methods.
- **Mitigation:** Never call `unserialize()` on untrusted input; use `allowed_classes => false` or an allowlist; verify with HMAC; prefer JSON.
- **References:** OWASP Deserialization Cheat Sheet, CWE-502.

### Gadget Chains

- **Attack:** Chaining multiple classes with dangerous magic methods to achieve RCE.
- **Mitigation:** Audit classes for dangerous `__destruct()`, `__wakeup()`, `__toString()`; use `allowed_classes`; keep dependencies updated.
- **Tools:** PHPGGC, ysoserial for PHP.

### Session File Injection

- **Attack:** Injecting serialized payloads into session files via headers or upload progress.
- **Mitigation:** Secure session storage permissions; use `session.use_strict_mode`; validate session data; use JSON handler.
- **References:** CVE-2010-3065, CVE-2007-1380.

### HMAC Verification

- **Purpose:** Ensure serialized data has not been tampered with.
- **Implementation:** `hash_hmac('sha256', $data, $secretKey)` + `hash_equals()`.
- **Limitation:** Requires secure key storage; does not protect against object injection if data is valid.

### Deprecated, Unsafe, or Version-Specific Features

| Feature | Status | Replacement |
|---|---|---|
| `Serializable` interface | Deprecated in PHP 8.1.0 | `__serialize()`/`__unserialize()` |
| `__sleep()` / `__wakeup()` | Still supported | `__serialize()`/`__unserialize()` |
| `wddx` session handler | Deprecated | `php_serialize` or JSON |
| `php_binary` session handler | Has security issues | `php_serialize` |
| `unserialize()` without `allowed_classes` | Dangerous | `allowed_classes => false` |
| `allowed_classes` on PHP < 7.0 | Not available | Upgrade PHP |
| Non-array `allowed_classes` in PHP 8.4.0 | Stricter handling | Use array or bool |

### References Links

- PHP: `serialize()` — https://www.php.net/manual/en/function.serialize.php
- PHP: `unserialize()` — https://www.php.net/manual/en/function.unserialize.php
- PHP: Object Serialization — https://www.php.net/manual/en/language.oop5.serialization.php
- PHP: Magic Methods — https://www.php.net/manual/en/language.oop5.magic.php
- PHP: `Serializable` Interface — https://www.php.net/manual/en/class.serializable.php
- PHP: `__serialize()` — https://www.php.net/manual/en/language.oop5.magic.php#object.serialize
- PHP: `__unserialize()` — https://www.php.net/manual/en/language.oop5.magic.php#object.unserialize
- PHP: `json_encode()` — https://www.php.net/manual/en/function.json-encode.php
- PHP: `json_decode()` — https://www.php.net/manual/en/function.json-decode.php
- PHP: `JsonSerializable` — https://www.php.net/manual/en/class.jsonserializable.php
- PHP: Session Configuration — https://www.php.net/manual/en/session.configuration.php
- PHP: `session.serialize_handler` — https://www.php.net/manual/en/session.configuration.php#ini.session.serialize-handler
- OWASP: Deserialization Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html
- OWASP: PHP Object Injection — https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection
- CWE-502: Deserialization of Untrusted Data — https://cwe.mitre.org/data/definitions/502.html
- HackTricks: PHP Deserialization — https://mintlify.wiki/HackTricks-wiki/hacktricks/pentesting-web/deserialization
- PHPGGC: PHP Gadget Chain Generator — https://github.com/ambionics/phpggc
- PHP RFC: `#[\NotSerializable]` Attribute — https://wiki.php.net/rfc/not_serializable
- CVE-2025-48951: Auth0-PHP Insecure Deserialization — https://nvd.nist.gov/vuln/detail/cve-2025-48951
- CVE-2026-33942: Saloon Insecure Deserialization — https://app.opencve.io/cve/CVE-2026-33942
- MediaWiki: Don't Use PHP Serialization — https://www.mediawiki.org/wiki/Manual:Coding_conventions/PHP