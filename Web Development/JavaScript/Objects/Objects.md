# JavaScript Object Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An object is a collection of related data and/or functionality, typically consisting of several variables (called properties) and functions (called methods) that belong to the object.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §6.1.7), an ECMAScript object is a collection of properties, each with zero or more attributes that determine how each property can be used. An object is an instance of the Object type, which is one of the fundamental data types in JavaScript. Objects are mutable, reference-based data structures. The Object constructor creates a new ordinary object when called as a constructor, and performs a type conversion when called as a function rather than as a constructor.

**Beginner-Friendly Explanation:** An object is like a filing cabinet. It holds information (called properties) and actions (called methods) related to one thing — like a person, a car, or a user account. Instead of having separate variables for `personName`, `personAge`, and `personGreet`, you can group them together in a single object.

### Key Characteristics

- **Collection of key-value pairs:** Objects store data as property name / property value pairs.
- **Reference type:** Objects are passed by reference, not by value.
- **Mutable:** Properties can be added, changed, or deleted after creation.
- **Prototype-based inheritance:** Objects inherit properties from their prototype.
- **Property names:** Must be strings or symbols; other types are coerced to strings.
- **Property values:** Can be any JavaScript value, including functions (which become methods).
- **Dynamic:** Properties can be added or removed at runtime.

### Prerequisites

- Basic JavaScript syntax (variables, data types, functions).
- Understanding of primitive values vs. reference values.
- Familiarity with functions and function expressions.
- Awareness of the `typeof` operator and data types.

### Related Programming Areas

- **Prototypes and inheritance:** Objects form the basis of JavaScript's inheritance model.
- **Classes (ES6+):** Syntactic sugar over prototype-based object creation.
- **JSON:** JavaScript Object Notation for data interchange.
- **DOM programming:** DOM elements are objects.
- **Module patterns:** Objects encapsulate state and behaviour.

---

## Objects

### Definitions

**Core Definition:** An object is a collection of related data and/or functionality, consisting of properties and methods.

**Technical Definition:** An ECMAScript object is a collection of properties each with zero or more attributes that determine how each property can be used. Each property is either a data property or an accessor property. A data property has a value plus attributes; an accessor property has a getter or setter function plus attributes. Nearly all objects in JavaScript are instances of Object; a typical object inherits properties (including methods) from Object.prototype.

**Beginner-Friendly Explanation:** An object is a way to group related things together. If you're building a website for a bookstore, you could have a `book` object that holds the title, author, price, and a method to display the book's information — all in one place.

### Purposes

- To group related data and functionality into a single unit.
- To model real-world entities (users, products, orders).
- To organise code into logical, reusable structures.
- To serve as the foundation for JavaScript's object-oriented programming.
- To enable namespacing and encapsulation.

### Syntax Rules and Structure

**General Syntax (Object literal):**
```js
const objectName = {
  property1: value1,
  property2: value2,
  methodName() { /* ... */ },
};
```

**Component Breakdown:**
- `objectName`: The variable holding the object.
- `property1: value1`: A property name / value pair.
- Each pair is separated by a comma.
- `methodName()`: A method defined with shorthand syntax.

**Constraints and Limitations:**
- Do not use an object literal at the beginning of a statement (the `{` is interpreted as a block).
- Property names must be strings or symbols; numbers are coerced to strings.
- Objects are reference types; assigning to a new variable copies the reference, not the object.

### Annotated Code Examples

#### Example 1: Creating and Using an Object

```js
// Step 1: Create an object with properties and methods.
const person = {
  name: ["Bob", "Smith"],
  age: 32,
  bio() {
    console.log(`${this.name[0]} ${this.name[1]} is ${this.age} years old.`);
  },
  introduceSelf() {
    console.log(`Hi! I'm ${this.name[0]}.`);
  },
};

// Step 2: Access properties and methods.
console.log(person.name);          // ["Bob", "Smith"]
console.log(person.name[0]);       // "Bob"
console.log(person.age);           // 32
person.bio();                      // "Bob Smith is 32 years old."
person.introduceSelf();            // "Hi! I'm Bob."
```

**Expected Output:**
```
[ 'Bob', 'Smith' ]
Bob
32
Bob Smith is 32 years old.
Hi! I'm Bob.
```

**Why This Output:** The object holds related data (name, age) and functionality (bio, introduceSelf). Dot notation accesses properties and calls methods.

#### Example 2: Objects as Namespaces

```js
// Step 1: Create a namespace-like object.
const MyApp = {
  users: [],
  addUser(name) {
    this.users.push({ name, id: this.users.length + 1 });
  },
  getUser(id) {
    return this.users.find((u) => u.id === id);
  },
};

// Step 2: Use the namespace.
MyApp.addUser("Alice");
MyApp.addUser("Bob");
console.log(MyApp.getUser(1)); // { name: "Alice", id: 1 }
console.log(MyApp.users.length); // 2
```

**Expected Output:**
```
{ name: 'Alice', id: 1 }
2
```

**Why This Output:** The object groups related data (`users`) and methods (`addUser`, `getUser`), preventing global namespace pollution.

### Real-World Cases

- **User profiles:** Storing user data and methods.
- **Configuration:** `const config = { apiUrl: "...", timeout: 5000 };`.
- **DOM elements:** Every DOM node is an object.
- **Module exports:** Objects as public APIs.

---

## Object Literals

### Definitions

**Core Definition:** An object literal is a list of zero or more pairs of property names and associated values, enclosed in curly braces `{}`.

**Technical Definition:** Per ECMA-262, an ObjectLiteral is a syntactic construct that evaluates to a newly created object. The object literal syntax is also called an object initializer. Each property definition in the literal adds a property to the object being created.

**Beginner-Friendly Explanation:** An object literal is the most common way to create an object — you just write curly braces with your key-value pairs inside.

### Purposes

- To create objects in a concise, readable syntax.
- To initialise objects with properties and methods in one expression.
- To serve as the default object creation syntax in modern JavaScript.

### Syntax Rules and Structure

```js
const obj = {
  property1: value1,
  property2: value2,
  "property with spaces": value3,
  7: "numeric key",
  nested: { inner: "value" },
};
```

**Syntax Rules:**
- Curly braces `{}` delimit the object.
- Property name / value pairs are separated by commas.
- Trailing commas are allowed.
- Property names can be identifiers, string literals, or numeric literals.
- Property names that are not valid identifiers must be quoted.
- The `__proto__` key sets the prototype (special case).

**Constraints and Limitations:**
- A trailing comma after the last property is allowed but not required.
- Property names are coerced to strings unless they are symbols.
- Object literals create a new object each time they are evaluated.

### Annotated Code Examples

#### Example 1: Basic Object Literal

```js
// Step 1: Create an object literal.
const car = {
  myCar: "Saturn",
  getCar: "Honda",
  special: "Toyota",
};

console.log(car.myCar);   // "Saturn"
console.log(car.getCar);  // "Honda"
console.log(car.special); // "Toyota"
```

**Expected Output:**
```
Saturn
Honda
Toyota
```

**Why This Output:** Each property is defined in the literal and accessed via dot notation.

#### Example 2: Nested Objects and Numeric Keys

```js
const car = {
  manyCars: {
    a: "Saab",
    b: "Jeep",
  },
  7: "Mazda",
};

console.log(car.manyCars.b); // "Jeep"
console.log(car[7]);          // "Mazda"
```

**Expected Output:**
```
Jeep
Mazda
```

**Why This Output:** Objects can be nested, and numeric keys are accessed with bracket notation.

### Real-World Cases

- **Configuration objects:** `const config = { host: "localhost", port: 3000 };`.
- **React props:** `{ name: "Alice", age: 30 }`.
- **API responses:** JSON parsed into object literals.

---

## Object Properties

### Definitions

**Core Definition:** A property is a name-value pair within an object, where the name is a string or symbol and the value can be any JavaScript value.

**Technical Definition:** Per ECMA-262, a property is a key-value pair. Each property has attributes that determine how it can be used. Data properties have a `[[Value]]` attribute and `[[Writable]]`, `[[Enumerable]]`, and `[[Configurable]]` attributes. Accessor properties have `[[Get]]` and `[[Set]]` functions instead of `[[Value]]` and `[[Writable]]`.

**Beginner-Friendly Explanation:** A property is like a labelled drawer in a filing cabinet. The label is the property name, and the contents are the property value.

### Purposes

- To store data associated with an object.
- To define the object's state.
- To enable property access and modification.

### Syntax Rules and Structure

```js
// Property definition
const obj = { name: "Alice", age: 30 };

// Property access
obj.name;       // "Alice"
obj["age"];     // 30

// Property assignment
obj.name = "Bob";
obj["age"] = 31;

// Property deletion
delete obj.name;
```

**Property Attributes:**
- `writable`: Can the value be changed?
- `enumerable`: Does it appear in `for...in` loops?
- `configurable`: Can it be deleted or modified?

### Annotated Code Examples

#### Example 1: Data Properties

```js
const user = {
  name: "Alice",
  age: 30,
};

// Step 1: Read property.
console.log(user.name);  // "Alice"

// Step 2: Write property.
user.age = 31;
console.log(user.age);   // 31

// Step 3: Add new property.
user.email = "alice@example.com";
console.log(user.email); // "alice@example.com"

// Step 4: Delete property.
delete user.age;
console.log(user.age);   // undefined
```

**Expected Output:**
```
Alice
31
alice@example.com
undefined
```

**Why This Output:** Data properties can be read, written, added, and deleted.

#### Example 2: Accessor Properties

```js
const person = {
  firstName: "Alice",
  lastName: "Smith",
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(value) {
    [this.firstName, this.lastName] = value.split(" ");
  },
};

console.log(person.fullName); // "Alice Smith"

person.fullName = "Bob Jones";
console.log(person.firstName); // "Bob"
console.log(person.lastName);  // "Jones"
```

**Expected Output:**
```
Alice Smith
Bob
Jones
```

**Why This Output:** Accessor properties use getters and setters to compute values on the fly.

### Real-World Cases

- **Form data:** `{ username: "...", password: "..." }`.
- **State management:** Redux state objects.
- **API resources:** User, Product, Order objects.

---

## Property Names

### Definitions

**Core Definition:** A property name (or key) is the identifier used to access a property value in an object.

**Technical Definition:** Per ECMA-262, property keys are either String values or Symbol values. When an object is created, property keys are coerced to strings unless they are symbols. Property names that are not valid JavaScript identifiers must be quoted in object literals.

**Beginner-Friendly Explanation:** Property names are the "labels" on the drawers of your filing cabinet. They tell you what's inside.

### Purposes

- To uniquely identify properties within an object.
- To enable property access via dot or bracket notation.
- To support dynamic property names (computed keys).

### Syntax Rules and Structure

```js
const obj = {
  // Valid identifier
  name: "value",

  // String literal (for names with spaces, etc.)
  "property with spaces": "value",

  // Numeric literal (coerced to string)
  42: "answer",

  // Computed property name
  ["key" + "Name"]: "computed",
};
```

**Syntax Rules:**
- Identifier names: letters, digits, `$`, `_`.
- String names: any string, must be quoted.
- Numeric names: coerced to strings.
- Symbol names: valid as keys.
- Computed names: expressions in `[]`.

**Constraints and Limitations:**
- Names that are not valid identifiers cannot be accessed with dot notation.
- Numeric keys are accessed with bracket notation.
- `__proto__` is a special key that sets the prototype.

### Annotated Code Examples

```js
const unusual = {
  "": "An empty string",
  "!": "Bang!",
  7: "Mazda",
};

// Step 1: Access with bracket notation.
console.log(unusual[""]);  // "An empty string"
console.log(unusual["!"]); // "Bang!"
console.log(unusual[7]);   // "Mazda"

// Step 2: Dot notation fails for these.
// unusual.;   // SyntaxError
// unusual.!;  // SyntaxError
```

**Expected Output:**
```
An empty string
Bang!
Mazda
```

**Why This Output:** Property names that are not valid identifiers require bracket notation.

### Real-World Cases

- **JSON keys:** Often contain spaces or special characters.
- **Dynamic keys:** `{ [userId]: userData }`.
- **Symbols:** Used for unique, non-string property keys.

---

## Property Values

### Definitions

**Core Definition:** A property value is the data stored under a property name in an object.

**Technical Definition:** Per ECMA-262, a property value is the `[[Value]]` attribute of a data property. It can be any ECMAScript language value: a primitive, an object, a function, or `undefined`. Accessor properties compute their values via getter functions instead of storing them directly.

**Beginner-Friendly Explanation:** Property values are the "contents" of a property — whatever you've stored under that name.

### Purposes

- To store data associated with an object.
- To enable reading and writing of object state.
- To support any JavaScript value type, including functions and objects.

### Syntax Rules and Structure

```js
const obj = {
  string: "text",
  number: 42,
  boolean: true,
  nullValue: null,
  undefinedValue: undefined,
  array: [1, 2, 3],
  nested: { inner: "value" },
  method() { return "I'm a function"; },
};
```

**Constraints and Limitations:**
- Property values can be any type.
- Functions stored as property values are called methods.
- `undefined` and `null` are valid property values.

### Annotated Code Examples

```js
const mixed = {
  name: "Alice",
  age: 30,
  hobbies: ["reading", "gaming"],
  address: { city: "Paris" },
  greet() { return `Hi, I'm ${this.name}`; },
};

console.log(mixed.name);      // "Alice"
console.log(mixed.hobbies[0]); // "reading"
console.log(mixed.address.city); // "Paris"
console.log(mixed.greet());   // "Hi, I'm Alice"
```

**Expected Output:**
```
Alice
reading
Paris
Hi, I'm Alice
```

**Why This Output:** Property values can be primitives, arrays, objects, or functions.

### Real-World Cases

- **User data:** Names, emails, avatars.
- **Product data:** Prices, descriptions, images.
- **Configuration:** Timeouts, URLs, feature flags.

---

## Property Access: Dot Notation

### Definitions

**Core Definition:** Dot notation accesses an object's property using a dot followed by the property name.

**Technical Definition:** In the `object.propertyName` syntax, the `propertyName` must be a valid JavaScript identifier, which can also be a reserved word. Dot notation provides direct property access when the property name is a static, valid identifier.

**Beginner-Friendly Explanation:** Dot notation is the most common way to access properties — just write the object name, a dot, and the property name.

### Purposes

- To access properties with valid identifier names.
- To provide concise, readable property access.
- To call methods on objects.

### Syntax Rules and Structure

```js
object.propertyName
object.methodName()
```

**Syntax Rules:**
- Property name must be a valid identifier.
- Reserved words are allowed (`obj.class` is valid).
- Numeric names are not allowed (`obj.1` is a SyntaxError).
- Private properties (`#name`) can only be accessed with dot notation within the class.

**Constraints and Limitations:**
- Cannot access properties with spaces or special characters.
- Cannot access computed/dynamic property names.

### Annotated Code Examples

```js
const person = { name: "Alice", age: 30 };

console.log(person.name); // "Alice"
console.log(person.age);  // 30

// Method call
const doc = {
  createElement(tag) { return `<${tag}>`; },
};
console.log(doc.createElement("div")); // "<div>"
```

**Expected Output:**
```
Alice
30
<div>
```

**Why This Output:** Dot notation provides direct access to valid identifier properties and methods.

### Real-World Cases

- **DOM manipulation:** `document.getElementById("app")`.
- **Chained access:** `user.profile.name`.
- **Method calls:** `array.push(item)`.

---

## Property Access: Bracket Notation

### Definitions

**Core Definition:** Bracket notation accesses an object's property using square brackets containing an expression that evaluates to the property name.

**Technical Definition:** In the `object[expression]` syntax, the `expression` should evaluate to a string or Symbol that represents the property's name. This allows access to properties with names that are not valid identifiers, as well as dynamic property access.

**Beginner-Friendly Explanation:** Bracket notation lets you access properties when you have the property name in a variable, or when the name has spaces or special characters.

### Purposes

- To access properties with dynamic names.
- To access properties with invalid identifier names.
- To access numeric keys.
- To use variables as property keys.

### Syntax Rules and Structure

```js
object[expression]
```

**Syntax Rules:**
- Expression must evaluate to a string or Symbol.
- Can use variables, function calls, or any expression.
- Whitespace before brackets is allowed.
- Works for all property names.

**Constraints and Limitations:**
- Beware of using square brackets with external input (security risk).
- Returns `undefined` for non-existent properties.

### Annotated Code Examples

```js
const person = { name: "Alice", age: 30, "favorite color": "blue" };

// Step 1: Variable as key.
const key = "name";
console.log(person[key]); // "Alice"

// Step 2: Expression as key.
console.log(person["a" + "ge"]); // 30

// Step 3: Property with spaces.
console.log(person["favorite color"]); // "blue"

// Step 4: Function call as key.
const getKey = () => "name";
console.log(person[getKey()]); // "Alice"
```

**Expected Output:**
```
Alice
30
blue
Alice
```

**Why This Output:** Bracket notation evaluates the expression inside brackets to determine the property name.

### Real-World Cases

- **Dynamic lookups:** `data[fieldName]`.
- **Internationalisation:** `translations[locale]`.
- **Iterating keys:** `for (const key in obj) { obj[key]; }`.

---

## Computed Property Names

### Definitions

**Core Definition:** Computed property names allow you to use an expression as a property name in an object literal, enclosed in square brackets.

**Technical Definition:** Per ES2015, object literals support computed property names. The expression inside the brackets is evaluated at object creation time, and the result (coerced to a string or symbol) becomes the property name.

**Beginner-Friendly Explanation:** Computed property names let you create properties with dynamic names — the name is calculated from a variable or expression.

### Purposes

- To create properties with dynamic names.
- To avoid repetitive code when property names are based on variables.
- To build objects from data.

### Syntax Rules and Structure

```js
const obj = {
  [expression]: value,
};
```

**Syntax Rules:**
- Expression is enclosed in square brackets.
- Evaluated at object creation time.
- Result is coerced to a string or symbol.
- Can be combined with method shorthand.

**Constraints and Limitations:**
- Expression must be evaluated at creation time.
- Cannot use shorthand property syntax with computed names directly.

### Annotated Code Examples

```js
const prop = "foo";
const key = "bar";

const obj = {
  [prop]: "value1",
  [key + "Key"]: "value2",
  ["method" + "Name"]() { return "called"; },
};

console.log(obj.foo);       // "value1"
console.log(obj.barKey);    // "value2"
console.log(obj.methodName()); // "called"
```

**Expected Output:**
```
value1
value2
called
```

**Why This Output:** The expressions in brackets are evaluated to determine property names.

### Real-World Cases

- **Dynamic form fields:** `{ [fieldName]: fieldValue }`.
- **Redux reducers:** `{ [action.type]: handler }`.
- **API response mapping:** `{ [id]: data }`.

---

## Shorthand Properties

### Definitions

**Core Definition:** Shorthand property syntax allows you to omit the value when the property name matches a variable name in scope.

**Technical Definition:** Per ES2015, in an object literal, `{ foo }` is shorthand for `{ foo: foo }`. The property name is taken from the variable name, and the value is the variable's current value.

**Beginner-Friendly Explanation:** If you have a variable `name` and you want to create an object with a property `name`, you can just write `{ name }` instead of `{ name: name }`.

### Purposes

- To reduce repetition when creating objects from variables.
- To improve readability when property names match variable names.
- To make object literals more concise.

### Syntax Rules and Structure

```js
const name = "Alice";
const age = 30;

// Shorthand
const person = { name, age };

// Equivalent to:
const person2 = { name: name, age: age };
```

**Syntax Rules:**
- The variable name becomes the property name.
- The variable value becomes the property value.
- Can be mixed with regular properties.

### Annotated Code Examples

```js
const firstName = "Alice";
const lastName = "Smith";
const age = 30;

// Step 1: Shorthand properties.
const person = { firstName, lastName, age };

console.log(person); // { firstName: "Alice", lastName: "Smith", age: 30 }

// Step 2: Mix shorthand with regular.
const mixed = { firstName, role: "admin" };
console.log(mixed.role); // "admin"
```

**Expected Output:**
```
{ firstName: 'Alice', lastName: 'Smith', age: 30 }
admin
```

**Why This Output:** The shorthand syntax creates properties with names matching the variable names and values from the variables.

### Real-World Cases

- **Returning objects from functions:** `return { x, y };`.
- **Building state objects:** `{ loading, data, error }`.
- **Destructuring round-trips:** `const { name } = person;`.

---

## Method Shorthand

### Definitions

**Core Definition:** Method shorthand allows you to define methods in object literals without the `function` keyword.

**Technical Definition:** Per ES2015, object literals support a shorthand method syntax where `methodName() {}` is equivalent to `methodName: function() {}`. The method is defined as a function-valued property.

**Beginner-Friendly Explanation:** Instead of writing `bio: function() {}`, you can write `bio() {}`.

### Purposes

- To define methods concisely.
- To improve readability of object literals.
- To make methods visually distinct from data properties.

### Syntax Rules and Structure

```js
const obj = {
  methodName() {
    // method body
  },
};
```

**Syntax Rules:**
- No `function` keyword.
- No colon after method name.
- Can be generators: `*methodName() {}`.
- Can be async: `async methodName() {}`.
- Can be combined with computed names: `[expr]() {}`.

### Annotated Code Examples

```js
const person = {
  name: ["Bob", "Smith"],
  age: 32,

  // Method shorthand
  bio() {
    console.log(`${this.name[0]} ${this.name[1]} is ${this.age} years old.`);
  },

  introduceSelf() {
    console.log(`Hi! I'm ${this.name[0]}.`);
  },
};

person.bio();          // "Bob Smith is 32 years old."
person.introduceSelf(); // "Hi! I'm Bob."
```

**Expected Output:**
```
Bob Smith is 32 years old.
Hi! I'm Bob.
```

**Why This Output:** Method shorthand provides a cleaner syntax for defining methods.

### Real-World Cases

- **Class-like objects:** Methods for objects created with literals.
- **React class components:** `render() {}`.
- **Utility objects:** `Math.max()`, `JSON.parse()`.

---

## Object Constructors

### Definitions

**Core Definition:** An object constructor is a function used with the `new` operator to create and initialise objects.

**Technical Definition:** Per ECMA-262, when a function is called as a constructor with `new`, a new object is created, the function's `[[Prototype]]` is set to the constructor's `prototype`, the function body executes with `this` bound to the new object, and the new object is returned (unless the function returns an object explicitly).

**Beginner-Friendly Explanation:** A constructor is like a blueprint for creating objects. You define what properties and methods every object of that type should have, and then you use `new` to stamp out instances.

### Purposes

- To create multiple objects with the same structure.
- To define a "template" for objects.
- To enable instance-specific data while sharing methods.
- To serve as the basis for ES6 classes.

### Syntax Rules and Structure

```js
function ConstructorName(param1, param2) {
  this.param1 = param1;
  this.param2 = param2;
  this.methodName = function() { /* ... */ };
}

const instance = new ConstructorName(arg1, arg2);
```

**Syntax Rules:**
- Constructor names conventionally start with a capital letter.
- `this` refers to the new object being created.
- No explicit `return` (unless returning an object).
- Called with `new`.

**Constraints and Limitations:**
- Calling a constructor without `new` may behave unexpectedly.
- Methods defined in the constructor are duplicated per instance (use `prototype` for sharing).
- `new` with arrow functions throws `TypeError`.

### Annotated Code Examples

#### Example 1: Basic Constructor

```js
// Step 1: Define a constructor function.
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.greet = function() {
    return `Hi, I'm ${this.name}`;
  };
}

// Step 2: Create instances.
const alice = new Person("Alice", 30);
const bob = new Person("Bob", 25);

console.log(alice.name);   // "Alice"
console.log(bob.age);      // 25
console.log(alice.greet()); // "Hi, I'm Alice"
```

**Expected Output:**
```
Alice
25
Hi, I'm Alice
```

**Why This Output:** The `new` operator creates a new object, binds `this` to it, and the constructor initialises its properties.

#### Example 2: Constructor with `prototype`

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

// Methods on prototype are shared by all instances.
Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};

const alice = new Person("Alice", 30);
console.log(alice.greet()); // "Hi, I'm Alice"
console.log(alice.hasOwnProperty("greet")); // false (inherited)
```

**Expected Output:**
```
Hi, I'm Alice
false
```

**Why This Output:** Methods on `Person.prototype` are shared by all instances, saving memory.

### Real-World Cases

- **Data models:** `User`, `Product`, `Order`.
- **DOM constructors:** `new Image()`, `new Audio()`.
- **Library classes:** `new Date()`, `new Map()`.

---

## `new Object()`

### Definitions

**Core Definition:** `new Object()` is the Object constructor invocation that creates a new empty object.

**Technical Definition:** Per ECMA-262 §20.1.1.1, when `Object` is called as a constructor (with `new`), it creates a new ordinary object with `%Object.prototype%` as its prototype. When called with an argument that is already an object, it returns that object unchanged. When called with a primitive, it wraps the primitive in an object.

**Beginner-Friendly Explanation:** `new Object()` creates a new, empty object. It's equivalent to `{}`.

### Purposes

- To create an empty object (rarely used; `{}` is preferred).
- To wrap primitives as objects (legacy pattern).
- To understand the Object constructor's behaviour.

### Syntax Rules and Structure

```js
new Object()           // Empty object
new Object(value)      // Wraps or returns value
Object(value)          // Type conversion (without new)
```

**Syntax Rules:**
- `new Object()` creates an empty object with `Object.prototype`.
- `new Object(obj)` returns `obj` unchanged if it's already an object.
- `new Object(primitive)` wraps the primitive.
- `Object(value)` without `new` performs type conversion.

**Constraints and Limitations:**
- `{}` is preferred for creating empty objects.
- Wrapper objects (`new String("x")`) are rarely needed.

### Annotated Code Examples

```js
// Step 1: Empty object.
const empty = new Object();
console.log(empty); // {}

// Step 2: Wrapping a primitive.
const wrapped = new Object(42);
console.log(typeof wrapped); // "object"
console.log(wrapped.valueOf()); // 42

// Step 3: Passing an object returns it unchanged.
const obj = { a: 1 };
const sameObj = new Object(obj);
console.log(sameObj === obj); // true
```

**Expected Output:**
```
{}
object
42
true
```

**Why This Output:** `new Object()` creates an empty object, wraps primitives, and returns objects unchanged.

### Real-World Cases

- **Legacy code:** Old codebases using `new Object()`.
- **Understanding the Object constructor:** Rarely used in modern code.
- **Type conversion:** `Object(value)` for explicit conversion.

---

## `Object.create()`

### Definitions

**Core Definition:** `Object.create()` creates a new object with a specified prototype object and optional property descriptors.

**Technical Definition:** Per ECMA-262 §20.1.2.2, `Object.create(proto, propertiesObject)` creates a new object with `proto` as its `[[Prototype]]`. The optional second argument is an object whose properties specify property descriptors for the new object. If `proto` is `null`, the new object has no prototype.

**Beginner-Friendly Explanation:** `Object.create()` lets you create an object that inherits from another object — you specify what the new object's "parent" should be.

### Purposes

- To create objects with a specific prototype.
- To implement prototype-based inheritance.
- To create objects with no prototype (`Object.create(null)`).
- To define properties with precise control (via descriptors).

### Syntax Rules and Structure

```js
Object.create(proto)
Object.create(proto, propertiesObject)
```

**Component Breakdown:**
- `proto`: The prototype object, or `null`.
- `propertiesObject` (optional): Property descriptors.

**Constraints and Limitations:**
- `proto` must be an object or `null`; other values throw `TypeError`.
- `Object.create(null)` creates an object with no inherited properties (useful as a map).
- Property descriptors are defined via `Object.defineProperties()`.

### Annotated Code Examples

#### Example 1: Basic Prototype Inheritance

```js
// Step 1: Define a prototype object.
const personProto = {
  greet() {
    return `Hi, I'm ${this.name}`;
  },
};

// Step 2: Create an object that inherits from personProto.
const alice = Object.create(personProto);
alice.name = "Alice";

console.log(alice.greet()); // "Hi, I'm Alice"
console.log(Object.getPrototypeOf(alice) === personProto); // true
```

**Expected Output:**
```
Hi, I'm Alice
true
```

**Why This Output:** `alice` inherits `greet` from `personProto` via the prototype chain.

#### Example 2: `Object.create(null)` for Pure Maps

```js
// Step 1: Create an object with no prototype.
const pureMap = Object.create(null);
pureMap.key = "value";

console.log(pureMap.key);        // "value"
console.log(pureMap.toString);   // undefined (no inherited methods)
console.log("toString" in pureMap); // false
```

**Expected Output:**
```
value
undefined
false
```

**Why This Output:** `Object.create(null)` creates an object with no prototype, so it doesn't inherit `Object.prototype` methods. This is useful for creating pure dictionaries.

### Real-World Cases

- **Prototypal inheritance:** `Object.create(animalProto)` for creating animals.
- **Pure dictionaries:** `Object.create(null)` for maps without prototype pollution.
- **Library internals:** Many libraries use `Object.create()` for inheritance.

---

## References

- MDN Web Docs – JavaScript object basics — https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Basics
- MDN Web Docs – Working with objects — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects
- MDN Web Docs – Object — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object
- MDN Web Docs – Object initializer — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer
- MDN Web Docs – Property accessors — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_accessors
- MDN Web Docs – Object.create() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create
- MDN Web Docs – new operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new
- MDN Web Docs – Grammar and types: Object literals — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#object_literals
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object Objects — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object-objects
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object Initializer — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-object-initializer
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Property Accessors — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-property-accessors
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.create — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.create
- MDN Web Docs – Enhanced Object Literals — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#enhanced_object_literals