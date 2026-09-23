# JSON Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
JSON (JavaScript Object Notation) is a lightweight, text-based, language-independent data interchange format used to represent structured data as collections of name/value pairs and ordered lists of values.

**Technical Definition**
JSON is a text format defined by ECMA-404 and IETF RFC 8259 (superseding RFC 4627). It is derived from a subset of the JavaScript Programming Language Standard ECMA-262 3rd Edition (December 1999) but is completely language-independent . JSON defines two universal data structures: objects (unordered collections of name/value pairs) and arrays (ordered collections of values). The value types are: string, number, object, array, true, false, and null .

**Beginner-Friendly Explanation**
JSON is a way to write data that both humans and computers can read easily. It looks a lot like how you'd write an object in JavaScript, but with stricter rules. It's the standard format for sending data between a browser and a server, or between different programs.

### Key Characteristics

- **Language-Independent**: JSON is a text format, not tied to any programming language .
- **Two Universal Structures**: Objects (key/value pairs) and arrays (ordered lists) map directly to data structures in virtually all modern programming languages .
- **Strict Syntax**: Fewer allowed values than JavaScript object literals (no functions, no `undefined`, no comments) .
- **Unicode Support**: JSON text is always Unicode; UTF-8 is the standard encoding for transmission .
- **No Cyclic References**: JSON cannot directly represent circular data structures .

### Prerequisites

- Basic understanding of data structures (objects, arrays, key/value pairs).
- Familiarity with JavaScript object literals (for comparison).
- A text editor and a way to run JavaScript (browser console or Node.js).

### Related Programming Areas

- **AJAX**: JSON is the preferred data format for asynchronous browser-server communication .
- **REST APIs**: JSON is the dominant payload format for modern web APIs.
- **Data Serialization**: Converting in-memory objects to a storable/transmittable format.
- **Configuration Files**: JSON is commonly used for application settings.

### Core Concepts / Features

1. JSON Syntax Rules vs. Standard JavaScript Objects
2. Valid JSON Data Types
3. Common Structural Patterns

---

## Core Concept 1: JSON Syntax Rules vs. Standard JavaScript Objects

### Definitions

**Core Definition**
JSON syntax is a strict subset of JavaScript object literal syntax, meaning every valid JSON text is also valid JavaScript, but not every JavaScript object literal is valid JSON.

**Technical Definition**
JSON was inspired by JavaScript object literals but imposes stricter rules for interoperability . The key differences include: all object keys must be double-quoted strings; only double quotes are allowed for strings; values are limited to JSON types (no functions, `undefined`, or symbols); numbers cannot use hexadecimal or octal notation; and trailing commas are not permitted . JSON is a text format for data interchange, while JavaScript object literals are a programming language construct for creating objects in code .

**Beginner-Friendly Explanation**
JSON looks like JavaScript objects but follows stricter rules. In JavaScript, you can write `{ name: "Alice" }` with unquoted keys and single quotes. In JSON, you must write `{ "name": "Alice" }` — keys always in double quotes, strings always in double quotes. Also, JSON can't contain functions or undefined values.

### Purposes

- To understand why JSON is stricter than JavaScript object literals.
- To avoid common syntax errors when writing JSON.
- To correctly convert between JavaScript objects and JSON strings.
- To ensure interoperability across programming languages.

### Syntax Rules and Structure

**Complete General Syntax**

```json
{
  "key": "value",
  "number": 42,
  "boolean": true,
  "nullValue": null,
  "array": [1, 2, 3],
  "nested": { "inner": "object" }
}
```

**Key Syntax Differences**

| Aspect | JSON | JavaScript Object Literal |
|---|---|---|
| Object keys | Must be double-quoted strings | May be unquoted identifiers |
| String quotes | Double quotes only | Single or double quotes |
| Trailing commas | Not allowed | Allowed in ES5+ |
| Comments | Not allowed | Allowed |
| Functions | Not allowed | Allowed |
| `undefined` | Not allowed | Allowed |
| Numbers | Decimal only | Decimal, hex, octal, binary |
| `NaN` / `Infinity` | Not allowed | Allowed |

**Component Breakdown**

- **Object**: Begins with `{`, ends with `}`. Contains zero or more name/value pairs separated by commas .
- **Name**: A string in double quotes, followed by a colon `:`.
- **Value**: A string, number, object, array, `true`, `false`, or `null` .
- **Array**: Begins with `[`, ends with `]`. Contains zero or more values separated by commas .

**Syntax Rules**

1. Object keys must be double-quoted strings .
2. Strings must be enclosed in double quotes; single quotes are invalid .
3. Numbers cannot use octal (`010`) or hexadecimal (`0xFF`) notation .
4. The values `undefined`, functions, and symbols are not valid JSON values .
5. `NaN` and `Infinity` are not valid JSON numbers .
6. Whitespace is allowed between any tokens but not inside tokens .
7. Trailing commas are not permitted.

**Constraints and Limitations**

- **No Functions**: JSON cannot represent executable code.
- **No `undefined`**: Values that are `undefined` in JavaScript are omitted during serialization .
- **No Comments**: JSON does not support comments, unlike JavaScript.
- **No Date Type**: Dates are typically represented as ISO 8601 strings.

### Multiple Annotated Complete Code Examples

**Example 1: Valid JSON vs. Invalid JavaScript Object Literal**

```javascript
// This is a valid JavaScript object literal
var jsObject = {
  name: "Alice",       // Unquoted key, double-quoted string
  age: 30,             // Unquoted key
  hobbies: ["reading", "gaming"],
  greet: function() { return "Hello"; }  // Function — not JSON
};

// This is the equivalent VALID JSON
// Note: keys are quoted, no function, no trailing comma
var jsonString = '{"name":"Alice","age":30,"hobbies":["reading","gaming"]}';

// Parse JSON into a JavaScript object
var parsed = JSON.parse(jsonString);
console.log(parsed.name);        // "Alice"
console.log(parsed.age);         // 30
console.log(parsed.hobbies[0]);  // "reading"

// Attempting to parse invalid JSON throws SyntaxError
try {
  JSON.parse('{ name: "Alice" }');  // Unquoted key — INVALID
} catch (e) {
  console.log("Error:", e.message); // "Unexpected token n in JSON at position 2"
}
```

**Expected Output (Console)**
```
Alice
30
reading
Error: Unexpected token n in JSON at position 2
```

**Why This Output Occurs**
The valid JSON string has quoted keys and no function. `JSON.parse()` successfully converts it to a JavaScript object. The invalid string with an unquoted key causes a `SyntaxError` .

---

**Example 2: Serialization Differences**

```javascript
// JavaScript object with values that JSON cannot represent
var obj = {
  name: "Alice",
  age: undefined,          // undefined — omitted in JSON
  greet: function() {},    // function — omitted in JSON
  score: NaN,              // NaN — becomes null in JSON
  active: true
};

// Serialize to JSON string
var json = JSON.stringify(obj);
console.log(json);
// Output: {"name":"Alice","score":null,"active":true}

// Note: age and greet are omitted
// NaN became null
// Trailing comma after active is NOT included
```

**Expected Output**
```
{"name":"Alice","score":null,"active":true}
```

**Why This Output Occurs**
`JSON.stringify()` omits `undefined` and function properties. `NaN` is converted to `null` because JSON has no representation for it .

### Real-World Cases

- **API Responses**: Servers return JSON strings that clients parse into objects.
- **Configuration**: JSON files store settings without executable code.
- **Data Storage**: JSON is used in NoSQL databases and local storage.

### References

- RFC 8259 — The JavaScript Object Notation (JSON) Data Interchange Format – https://datatracker.ietf.org/doc/html/rfc8259
- ECMA-404 — The JSON Data Interchange Syntax – https://www.ecma-international.org/publications-and-standards/standards/ecma-404/
- MDN Web Docs — JSON.parse() – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
- MDN Web Docs — JSON.stringify() – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify

---

## Core Concept 2: Valid JSON Data Types

### Definitions

**Core Definition**
JSON supports six data types: objects, arrays, strings, numbers, booleans, and null.

**Technical Definition**
JSON defines seven value types: string, number, object, array, `true`, `false`, and `null` . These map to the two universal data structures: objects (collections of name/value pairs) and arrays (ordered lists of values) . Primitive types (string, number, boolean, null) represent atomic values, while objects and arrays provide structure and nesting.

**Beginner-Friendly Explanation**
JSON can hold six kinds of values: text in quotes (strings), numbers, true/false (booleans), nothing (null), lists in square brackets (arrays), and key-value collections in curly braces (objects).

### Purposes

- To correctly represent data in JSON format.
- To choose the appropriate type for each data element.
- To understand what values are valid when writing JSON.
- To convert between JSON and native data types.

### Syntax Rules and Structure

**Complete General Syntax for Each Type**

```json
{
  "string": "Hello, World!",
  "number": 42,
  "decimal": 3.14,
  "booleanTrue": true,
  "booleanFalse": false,
  "nullValue": null,
  "array": [1, "two", false, null],
  "object": { "nested": "value" }
}
```

**Data Type Reference**

| Type | Example | Notes |
|---|---|---|
| String | `"hello"` | Double quotes; backslash escapes  |
| Number | `42`, `-3.14`, `1e10` | Decimal only; no hex/octal  |
| Boolean | `true`, `false` | Lowercase only |
| Null | `null` | Lowercase only |
| Object | `{"key": "value"}` | Unordered name/value pairs  |
| Array | `[1, 2, 3]` | Ordered list of values  |

**Component Breakdown**

- **String**: Sequence of Unicode characters wrapped in double quotes. Escapes include `\"`, `\\`, `\n`, `\t`, `\uXXXX` .
- **Number**: Base-10 representation; may have minus sign, decimal point, and exponent (`e` or `E`) .
- **Object**: `{` followed by zero or more `"name": value` pairs separated by `,`, ending with `}` .
- **Array**: `[` followed by zero or more values separated by `,`, ending with `]` .

**Syntax Rules**

1. Strings must use double quotes; single quotes are invalid .
2. Numbers cannot have leading zeros (except `0` itself) .
3. `NaN` and `Infinity` are not valid JSON numbers .
4. Object keys must be strings (double-quoted) .
5. Array order is significant; object member order is insignificant .
6. All values can be nested arbitrarily (objects in arrays, arrays in objects, etc.) .

**Constraints and Limitations**

- **No Date Type**: Dates must be strings (ISO 8601 recommended).
- **No Undefined**: `undefined` is not a JSON value.
- **No Comments**: JSON does not support comments.
- **Precision**: Large numbers may lose precision when parsed as JavaScript numbers .

### Multiple Annotated Complete Code Examples

**Example 1: All JSON Data Types**

```json
{
  "stringValue": "Hello, World!",
  "numberValue": 42,
  "decimalValue": 3.14159,
  "negativeValue": -17,
  "exponentValue": 1.5e10,
  "booleanTrue": true,
  "booleanFalse": false,
  "nullValue": null,
  "arrayValue": [1, "two", true, null, { "nested": "object" }],
  "objectValue": {
    "name": "Alice",
    "age": 30,
    "active": true
  }
}
```

**Expected Output**
This is a valid JSON document containing all six JSON data types.

**Why This Output Occurs**
Every value conforms to JSON syntax: strings are double-quoted, numbers are decimal, booleans and null are lowercase, arrays and objects use the correct delimiters .

---

**Example 2: Parsing and Type Checking**

```javascript
var jsonText = '{"name":"Alice","age":30,"active":true,"hobbies":["reading","gaming"],"address":null}';

var obj = JSON.parse(jsonText);

console.log(typeof obj);              // "object"
console.log(typeof obj.name);         // "string"
console.log(typeof obj.age);          // "number"
console.log(typeof obj.active);       // "boolean"
console.log(Array.isArray(obj.hobbies)); // true
console.log(obj.address);             // null

// Accessing nested array elements
console.log(obj.hobbies[0]);          // "reading"
console.log(obj.hobbies[1]);          // "gaming"
```

**Expected Output (Console)**
```
object
string
number
boolean
true
null
reading
gaming
```

**Why This Output Occurs**
`JSON.parse()` converts JSON text into JavaScript values. The `typeof` operator confirms the JavaScript types, and `Array.isArray()` confirms the array .

### Real-World Cases

- **User Profiles**: `{"name": "Alice", "age": 30, "active": true}`.
- **Product Data**: `{"id": 101, "price": 19.99, "tags": ["electronics", "sale"]}`.
- **Settings**: `{"theme": "dark", "notifications": false, "volume": null}`.

### References

- MDN Web Docs — JSON – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON
- JSON.org — Introducing JSON – https://www.json.org/json-en.html
- RFC 8259 — Values – https://datatracker.ietf.org/doc/html/rfc8259#section-3

---

## Core Concept 3: Common Structural Patterns

### Definitions

**Core Definition**
Common structural patterns in JSON are recurring ways of organising objects and arrays to represent real-world data relationships.

**Technical Definition**
JSON's two structures (objects and arrays) can be nested arbitrarily to represent complex data. Objects map to records, structs, dictionaries, or hash tables; arrays map to lists, vectors, or sequences . Because these structures compose, JSON can represent trees and other hierarchical data . Common patterns include: flat objects for records, arrays of objects for collections, nested objects for hierarchical relationships, and mixed arrays for heterogeneous data .

**Beginner-Friendly Explanation**
JSON data usually follows a few patterns. A single item is an object. A list of items is an array of objects. Details inside an item are nested objects. A list of tags or simple values is an array of strings or numbers.

### Purposes

- To recognise and apply standard data organisation patterns.
- To design JSON structures for APIs and data storage.
- To parse and navigate nested JSON efficiently.
- To represent real-world relationships (one-to-many, hierarchical).

### Syntax Rules and Structure

**Complete General Syntax for Common Patterns**

**Pattern 1: Flat Object (Single Record)**
```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

**Pattern 2: Array of Objects (Collection)**
```json
[
  { "id": 1, "name": "Alice" },
  { "id": 2, "name": "Bob" },
  { "id": 3, "name": "Carol" }
]
```

**Pattern 3: Nested Object (Hierarchy)**
```json
{
  "id": 1,
  "name": "Alice",
  "address": {
    "street": "123 Main St",
    "city": "Springfield",
    "coordinates": {
      "lat": 39.7817,
      "lng": -89.6501
    }
  }
}
```

**Pattern 4: Object with Array Property (One-to-Many)**
```json
{
  "id": 1,
  "name": "Alice",
  "hobbies": ["reading", "gaming", "hiking"],
  "orders": [
    { "orderId": 101, "total": 25.99 },
    { "orderId": 102, "total": 14.50 }
  ]
}
```

**Component Breakdown**

- **Object**: Used for records with named fields .
- **Array**: Used for ordered lists and collections .
- **Nesting**: Objects and arrays can contain each other arbitrarily .
- **Key Naming**: Use descriptive, consistent names (camelCase or snake_case).

**Syntax Rules**

1. Arrays preserve order; object member order is insignificant .
2. Nested structures can be arbitrarily deep.
3. Keys must be unique within an object.
4. Arrays can contain mixed types, though consistent types are usually clearer .

**Constraints and Limitations**

- **No Cyclic References**: JSON cannot represent objects that reference each other circularly .
- **No Schema Enforcement**: JSON itself does not validate structure; JSON Schema is a separate standard.
- **Flat vs. Nested Trade-off**: Deeply nested structures can be harder to query.

### Multiple Annotated Complete Code Examples

**Example 1: Array of Objects (Collection)**

```javascript
var usersJson = '[{"id":1,"name":"Alice","active":true},{"id":2,"name":"Bob","active":false},{"id":3,"name":"Carol","active":true}]';

var users = JSON.parse(usersJson);

// Iterate over collection
users.forEach(function(user) {
  console.log(user.name + " is " + (user.active ? "active" : "inactive"));
});

// Filter active users
var activeUsers = users.filter(function(u) { return u.active; });
console.log("Active count:", activeUsers.length);
```

**Expected Output (Console)**
```
Alice is active
Bob is inactive
Carol is active
Active count: 2
```

**Why This Output Occurs**
The JSON array contains three objects. After parsing, native array methods (`forEach`, `filter`) can be used on the collection .

---

**Example 2: Nested Objects (Hierarchy)**

```javascript
var companyJson = '{"name":"TechCorp","departments":[{"name":"Engineering","employees":[{"name":"Alice","role":"Developer"},{"name":"Bob","role":"Designer"}]},{"name":"Sales","employees":[{"name":"Carol","role":"Manager"}]}]}';

var company = JSON.parse(companyJson);

// Access nested data
console.log("Company:", company.name);
company.departments.forEach(function(dept) {
  console.log("Department:", dept.name);
  dept.employees.forEach(function(emp) {
    console.log("  - " + emp.name + " (" + emp.role + ")");
  });
});
```

**Expected Output (Console)**
```
Company: TechCorp
Department: Engineering
  - Alice (Developer)
  - Bob (Designer)
Department: Sales
  - Carol (Manager)
```

**Why This Output Occurs**
The nested structure represents a company with departments, each containing employees. Nested loops traverse the hierarchy .

---

**Example 3: Object with Array of Objects (One-to-Many)**

```javascript
var orderJson = '{"orderId":1001,"customer":"Alice","items":[{"product":"Laptop","qty":1,"price":999.99},{"product":"Mouse","qty":2,"price":29.99},{"product":"Keyboard","qty":1,"price":79.99}]}';

var order = JSON.parse(orderJson);

console.log("Order #" + order.orderId + " for " + order.customer);
var total = 0;
order.items.forEach(function(item) {
  var subtotal = item.qty * item.price;
  total += subtotal;
  console.log("  " + item.qty + "x " + item.product + " @ $" + item.price + " = $" + subtotal.toFixed(2));
});
console.log("Total: $" + total.toFixed(2));
```

**Expected Output (Console)**
```
Order #1001 for Alice
  1x Laptop @ $999.99 = $999.99
  2x Mouse @ $29.99 = $59.98
  1x Keyboard @ $79.99 = $79.99
Total: $1139.96
```

**Why This Output Occurs**
The object contains an array of item objects. Iterating over the array and accessing each item's properties calculates the total .

### Real-World Cases

- **REST API Responses**: Collections of resources (users, products, posts) as arrays of objects.
- **Configuration Files**: Nested settings with sections and subsections.
- **Data Export**: Database records serialised as JSON arrays for backup.
- **Real-Time Feeds**: Streams of JSON objects (e.g., Twitter firehose) .

### References

- JSON.org — Introducing JSON – https://www.json.org/json-en.html
- MDN Web Docs — JSON – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON
- IETF — RFC 8259 – https://datatracker.ietf.org/doc/html/rfc8259

---

## Summary Comparison Table

| Aspect | JSON | JavaScript Object Literal |
|---|---|---|
| **Purpose** | Data interchange | Code object creation |
| **Keys** | Must be double-quoted | May be unquoted |
| **Strings** | Double quotes only | Single or double quotes |
| **Functions** | Not allowed | Allowed |
| **undefined** | Not allowed | Allowed |
| **Comments** | Not allowed | Allowed |
| **Number formats** | Decimal only | Decimal, hex, octal, binary |
| **NaN / Infinity** | Not allowed | Allowed |
| **Cyclic references** | Not representable | Representable |

---

## Important Notes on Version-Specific Behaviour and Limitations

1. **JSON is a Subset of JavaScript**: Every valid JSON text is valid JavaScript, but not vice versa .
2. **JSON.parse() and JSON.stringify()**: Available in all modern browsers since ES5 (2009), baseline widely available since July 2015 .
3. **No Schema Validation**: JSON itself does not validate structure; use JSON Schema for validation.
4. **Number Precision**: Large integers may lose precision when parsed as JavaScript `Number`; use strings or `BigInt` for arbitrary precision .
5. **`__proto__` Key**: `JSON.parse()` handles `__proto__` differently from JavaScript object literals .
6. **Encoding**: JSON text is always Unicode; UTF-8 is standard for transmission .

---

## References

- RFC 8259 — The JavaScript Object Notation (JSON) Data Interchange Format – https://datatracker.ietf.org/doc/html/rfc8259
- ECMA-404 — The JSON Data Interchange Syntax – https://www.ecma-international.org/publications-and-standards/standards/ecma-404/
- JSON.org — Introducing JSON – https://www.json.org/json-en.html
- JSON.org — JSON: The Fat-Free Alternative to XML – https://www.json.org/fatfree.html
- MDN Web Docs — JSON.parse() – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
- MDN Web Docs — JSON.stringify() – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify
- MDN Web Docs — JSON – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON
- Oracle — Introduction to JSON – https://docs.oracle.com/javaee/7/tutorial/jsonp001.htm
- Oracle — JSON Developer's Guide – https://docs.oracle.com/en/database/oracle/oracle-database/12.2/adjsn/json-developers-guide.pdf
- Stack Overflow — Difference between JSON and Object Literal Notation – https://stackoverflow.com/questions/2904131/what-is-the-difference-between-json-and-object-literal-notation