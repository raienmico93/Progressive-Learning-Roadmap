# JavaScript Getters and Setters — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Getters and setters are special methods that intercept property access and assignment, allowing computed values, validation, side effects, and encapsulation.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §6.1.7.1), a getter is an accessor property whose `[[Get]]` field is a function, and a setter is an accessor property whose `[[Set]]` field is a function. Accessor properties do not have a `[[Value]]` or `[[Writable]]` attribute; they instead provide `[[Get]]` and/or `[[Set]]` functions. When a property access is performed, the engine invokes `[[Get]]`; when an assignment is performed, the engine invokes `[[Set]]`. Accessor properties are defined via object literals (`get`/`set` syntax), `Object.defineProperty()`, `Object.defineProperties()`, or `class` bodies.

**Beginner-Friendly Explanation:** Normally, when you read `obj.x`, JavaScript just gives you the value stored there. But sometimes you want something smarter to happen — like computing a value on the fly, validating input, or logging access. Getters and setters let you hook into property reads and writes so you can run custom code when someone accesses or changes a property. They look like normal properties from the outside but behave like functions inside.

### Key Characteristics

- **Accessor properties:** Defined by `get`/`set` functions instead of a value.
- **Transparent syntax:** Called like normal properties (`obj.x`, not `obj.x()`).
- **Computed on the fly:** Getters compute values each time they are accessed.
- **Validation and side effects:** Setters can validate input and trigger side effects.
- **Defined in literals, classes, and descriptors.**
- **Not enumerable by default in classes:** Methods (including accessors) in classes are non-enumerable.
- **Can be computed:** Property names can be computed expressions.
- **No `writable` attribute:** Accessor properties use `get`/`set` instead.

### Prerequisites

- Basic JavaScript syntax (objects, properties, functions).
- Understanding of property descriptors and attributes.
- Familiarity with `this` and method invocation.
- Awareness of object literals and classes.
- Basic knowledge of prototype chains.

### Related Programming Areas

- **Encapsulation:** Hiding internal state behind accessors.
- **Validation:** Enforcing constraints on property values.
- **Computed properties:** Deriving values from other properties.
- **Reactive programming:** Frameworks use getters/setters for reactivity (Vue, MobX).
- **Proxy and Reflect:** Complements to accessor-based metaprogramming.
- **Class design:** Encapsulation in modern JavaScript classes.

---

## Getters

### Definitions

**Core Definition:** A getter is a function that is invoked when a property is read, returning a computed or stored value.

**Technical Definition:** Per ECMA-262, a getter is an accessor property whose `[[Get]]` internal method is a function. When `[[Get]]` is invoked on a property, the function is called with `this` bound to the object, and its return value becomes the result of the property access. A getter can be defined without a matching setter, making the property effectively read-only.

**Beginner-Friendly Explanation:** A getter is a property that runs a function every time you read it. You write `obj.value`, and behind the scenes JavaScript calls a function that returns the value.

### Purposes

- To compute a property value on demand.
- To provide read-only access to internal state.
- To derive values from other properties.
- To log or track property access.
- To lazily compute expensive values.

### Syntax Rules and Structure

**Object literal:**
```js
const obj = {
  get propName() {
    // return value
  },
};
```

**Class:**
```js
class MyClass {
  get propName() {
    // return value
  }
}
```

**Property descriptor:**
```js
Object.defineProperty(obj, "propName", {
  get() { return value; },
  enumerable: true,
  configurable: true,
});
```

**Computed:**
```js
const key = "dynamicKey";
const obj = {
  get [key]() { return "computed"; },
};
```

**Component Breakdown:**
- `get`: The keyword introducing a getter.
- `propName`: The property name (identifier, string, or computed).
- Body: The function to run on access.

**Syntax Rules:**
- A getter takes no parameters.
- Must not be defined with `function` keyword inside object literals.
- Can be defined alongside a setter.
- Can be defined in classes, object literals, and via `defineProperty`.
- Cannot have the same name as a data property in the same object literal.

**Constraints and Limitations:**
- Getters cannot take parameters.
- A getter-only property cannot be assigned (silently fails in non-strict, throws in strict).
- Getters can cause infinite loops if they reference themselves.
- Getters add overhead to every property access.

### Annotated Code Examples

#### Example 1: Basic Getter

```js
// Step 1: Define an object with a getter.
const person = {
  firstName: "Alice",
  lastName: "Smith",

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
};

// Step 2: Access the getter like a property.
console.log(person.fullName); // "Alice Smith"

// Step 3: Change the underlying data.
person.firstName = "Bob";
console.log(person.fullName); // "Bob Smith"
```

**Expected Output:**
```
Alice Smith
Bob Smith
```

**Why This Output:** The getter computes `fullName` on each access using the current `firstName` and `lastName`.

#### Example 2: Read-Only Computed Property

```js
const circle = {
  radius: 5,

  get area() {
    return Math.PI * this.radius ** 2;
  },

  get circumference() {
    return 2 * Math.PI * this.radius;
  },
};

console.log(circle.area.toFixed(2));          // "78.54"
console.log(circle.circumference.toFixed(2)); // "31.42"

circle.radius = 10;
console.log(circle.area.toFixed(2));          // "314.16"
```

**Expected Output:**
```
78.54
31.42
314.16
```

**Why This Output:** The getters compute values based on the current `radius`, so changing `radius` updates the computed values.

#### Example 3: Getter in a Class

```js
class Temperature {
  constructor(celsius) {
    this._celsius = celsius;
  }

  get celsius() {
    return this._celsius;
  }

  get fahrenheit() {
    return this._celsius * 9 / 5 + 32;
  }
}

const temp = new Temperature(25);
console.log(temp.celsius);    // 25
console.log(temp.fahrenheit); // 77
```

**Expected Output:**
```
25
77
```

**Why This Output:** Class getters provide computed access to internal state.

### Real-World Cases

- **Vue reactivity:** Getters track dependencies.
- **Derived values:** Full names, totals, formatted strings.
- **Read-only properties:** `Math.PI`, `array.length`.
- **Lazy computation:** Computing expensive values only when accessed.

---

## Setters

### Definitions

**Core Definition:** A setter is a function that is invoked when a property is assigned, allowing validation, transformation, or side effects.

**Technical Definition:** Per ECMA-262, a setter is an accessor property whose `[[Set]]` internal method is a function. When `[[Set]]` is invoked on a property, the function is called with `this` bound to the object and the assigned value as its only argument. A setter can be defined without a matching getter, making the property effectively write-only.

**Beginner-Friendly Explanation:** A setter is a property that runs a function every time you assign to it. You write `obj.value = 10`, and behind the scenes JavaScript calls a function that processes the value.

### Purposes

- To validate input before storing.
- To transform values on assignment.
- To trigger side effects (e.g., DOM updates, logging).
- To maintain invariants on internal state.
- To implement computed properties with custom storage.

### Syntax Rules and Structure

**Object literal:**
```js
const obj = {
  set propName(value) {
    // handle assignment
  },
};
```

**Class:**
```js
class MyClass {
  set propName(value) {
    // handle assignment
  }
}
```

**Property descriptor:**
```js
Object.defineProperty(obj, "propName", {
  set(value) { /* ... */ },
  enumerable: true,
  configurable: true,
});
```

**Component Breakdown:**
- `set`: The keyword introducing a setter.
- `propName`: The property name.
- `value`: The parameter receiving the assigned value.

**Syntax Rules:**
- A setter takes exactly one parameter.
- Can be defined alongside a getter.
- Can be defined in classes, object literals, and via `defineProperty`.
- Cannot have the same name as a data property in the same object literal.

**Constraints and Limitations:**
- Setter-only properties cannot be read (returns `undefined`).
- Setters add overhead to every assignment.
- Setters can cause infinite loops if they assign to themselves.
- Setter assignment can silently fail on getter-only properties in non-strict mode.

### Annotated Code Examples

#### Example 1: Basic Setter

```js
// Step 1: Define an object with a setter.
const person = {
  _name: "",

  set name(value) {
    if (typeof value !== "string") {
      throw new TypeError("Name must be a string");
    }
    this._name = value.trim();
  },

  get name() {
    return this._name;
  },
};

// Step 2: Assign through the setter.
person.name = "  Alice  ";
console.log(person.name); // "Alice" (trimmed)

// Step 3: Invalid input throws.
try {
  person.name = 42;
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
Alice
TypeError
```

**Why This Output:** The setter validates and transforms the input before storing.

#### Example 2: Setter with Side Effects

```js
const user = {
  _age: 0,

  set age(value) {
    if (value < 0) throw new RangeError("Age cannot be negative");
    this._age = value;
    console.log(`Age updated to ${value}`);
  },

  get age() {
    return this._age;
  },
};

user.age = 30; // "Age updated to 30"
user.age = 25; // "Age updated to 25"
```

**Expected Output:**
```
Age updated to 30
Age updated to 25
```

**Why This Output:** The setter logs each assignment as a side effect.

#### Example 3: Setter in a Class

```js
class Temperature {
  constructor(celsius) {
    this._celsius = celsius;
  }

  get celsius() {
    return this._celsius;
  }

  set celsius(value) {
    if (value < -273.15) {
      throw new RangeError("Temperature below absolute zero");
    }
    this._celsius = value;
  }

  get fahrenheit() {
    return this._celsius * 9 / 5 + 32;
  }

  set fahrenheit(value) {
    this._celsius = (value - 32) * 5 / 9;
  }
}

const temp = new Temperature(25);
temp.fahrenheit = 100;
console.log(temp.celsius); // 37.77777777777778

try {
  temp.celsius = -300;
} catch (err) {
  console.log(err.name); // "RangeError"
}
```

**Expected Output:**
```
37.77777777777778
RangeError
```

**Why This Output:** Setters validate and convert values before storing.

### Real-World Cases

- **Form validation:** Validating input before storing.
- **Vue reactivity:** Setters trigger re-renders.
- **MobX:** Observables use setters.
- **DOM properties:** `element.innerHTML`, `element.value`.

---

## `get` Syntax

### Definitions

**Core Definition:** The `get` syntax is the keyword used to define a getter in an object literal, class body, or property descriptor.

**Technical Definition:** Per ECMA-262, the `get` keyword in a `MethodDefinition` or `PropertyDefinition` introduces a getter accessor. The grammar production is `get PropertyName ( ) { FunctionBody }`. The method's `[[Kind]]` is `"get"`, and the property is created with `[[Get]]` set to the function and `[[Set]]` set to `undefined` (unless a setter is also defined).

**Beginner-Friendly Explanation:** `get` is the keyword you write before a property name to turn it into a getter.

### Purposes

- To define read accessors in object literals.
- To define read accessors in classes.
- To define read accessors via descriptors (though descriptors use `get:` as a key).

### Syntax Rules and Structure

```js
const obj = {
  get propName() { /* ... */ },
};

class MyClass {
  get propName() { /* ... */ }
}

const key = "dynamic";
const obj2 = {
  get [key]() { return "computed"; },
};
```

**Syntax Rules:**
- `get` is followed by the property name and `()`.
- No `function` keyword.
- The body must return a value (otherwise `undefined`).
- Can be combined with a `set` for the same property name.
- In classes, getters are non-enumerable by default.

**Constraints and Limitations:**
- Cannot be used with the `function` keyword.
- Cannot have parameters.
- Cannot be defined as an arrow function.
- `get` is a contextual keyword (can be a property name elsewhere).

### Annotated Code Examples

#### Example 1: Getter in Object Literal

```js
const obj = {
  _value: 42,
  get value() {
    return this._value;
  },
};

console.log(obj.value); // 42
```

**Expected Output:**
```
42
```

**Why This Output:** The `get` syntax defines a getter that returns `_value`.

#### Example 2: Computed Getter

```js
const prefix = "user";

const data = {
  userName: "Alice",
  userAge: 30,
  get [`${prefix}Info`]() {
    return `${this.userName} (${this.userAge})`;
  },
};

console.log(data.userInfo); // "Alice (30)"
```

**Expected Output:**
```
Alice (30)
```

**Why This Output:** The computed getter name combines `prefix` with `"Info"`.

### Real-World Cases

- **Vue computed properties:** Defined via `computed: { fullName() { ... } }`.
- **Class encapsulation:** Read-only class properties.
- **Lazy loading:** Computing values on demand.

---

## `set` Syntax

### Definitions

**Core Definition:** The `set` syntax is the keyword used to define a setter in an object literal, class body, or property descriptor.

**Technical Definition:** Per ECMA-262, the `set` keyword in a `MethodDefinition` introduces a setter accessor. The grammar production is `set PropertyName ( PropertySetParameterList ) { FunctionBody }`. The method's `[[Kind]]` is `"set"`, and the property is created with `[[Set]]` set to the function and `[[Get]]` set to `undefined` (unless a getter is also defined).

**Beginner-Friendly Explanation:** `set` is the keyword you write before a property name to turn it into a setter.

### Purposes

- To define write accessors in object literals.
- To define write accessors in classes.
- To define write accessors via descriptors.

### Syntax Rules and Structure

```js
const obj = {
  set propName(value) { /* ... */ },
};

class MyClass {
  set propName(value) { /* ... */ }
}
```

**Syntax Rules:**
- `set` is followed by the property name and `(value)`.
- Exactly one parameter.
- No `function` keyword.
- Can be combined with a `get` for the same property name.
- In classes, setters are non-enumerable by default.

**Constraints and Limitations:**
- Cannot have multiple parameters.
- Cannot be used with `function` keyword.
- Cannot be an arrow function.
- Assigning to a setter-only property then reading it returns `undefined`.

### Annotated Code Examples

#### Example 1: Setter in Object Literal

```js
const obj = {
  _value: 0,
  set value(v) {
    this._value = v * 2;
  },
  get value() {
    return this._value;
  },
};

obj.value = 10;
console.log(obj.value); // 20
```

**Expected Output:**
```
20
```

**Why This Output:** The setter doubles the value before storing it.

#### Example 2: Validation Setter

```js
const user = {
  _email: "",
  set email(value) {
    if (!value.includes("@")) {
      throw new TypeError("Invalid email");
    }
    this._email = value;
  },
  get email() {
    return this._email;
  },
};

user.email = "alice@example.com";
console.log(user.email); // "alice@example.com"

try {
  user.email = "invalid";
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
alice@example.com
TypeError
```

**Why This Output:** The setter validates the email before storing it.

### Real-World Cases

- **Vue reactivity:** Setters trigger updates.
- **Form validation:** Validating on assignment.
- **DOM properties:** `element.innerHTML = "..."`.

---

## Computed Getters

### Definitions

**Core Definition:** Computed getters (and setters) are accessors whose property names are determined by an expression rather than a fixed identifier.

**Technical Definition:** Per ECMA-262, the `PropertyName` in a `MethodDefinition` can be a `ComputedPropertyName` of the form `[ AssignmentExpression ]`. The expression is evaluated at object/class definition time, and its result (coerced to a string or symbol) becomes the property name.

**Beginner-Friendly Explanation:** A computed getter is a getter whose name is calculated from an expression — useful when you need dynamic property names.

### Purposes

- To define accessors with dynamic names.
- To avoid repetition when names follow a pattern.
- To use symbols as accessor keys.

### Syntax Rules and Structure

```js
const key = "dynamic";
const obj = {
  get [key]() { return "value"; },
};

const sym = Symbol("meta");
const obj2 = {
  get [sym]() { return "symbol value"; },
};
```

**Constraints and Limitations:**
- Expression must be evaluated at definition time.
- Cannot combine with shorthand property syntax.
- Result is coerced to a string or symbol.

### Annotated Code Examples

```js
const prefix = "full";

const person = {
  firstName: "Alice",
  lastName: "Smith",
  get [`${prefix}Name`]() {
    return `${this.firstName} ${this.lastName}`;
  },
};

console.log(person.fullName); // "Alice Smith"
```

**Expected Output:**
```
Alice Smith
```

**Why This Output:** The getter name is computed from `prefix + "Name"`.

### Real-World Cases

- **Localisation:** `get ["name_" + locale]()`.
- **Framework internals:** Vue uses computed getter names.
- **Symbol keys:** Accessing via well-known symbols.

---

## Use Cases

### Definitions

**Core Definition:** Use cases for getters and setters are the practical scenarios where computed property access or assignment provides value.

**Technical Definition:** Getters and setters are applicable when property access needs to trigger computation, validation, side effects, or encapsulation. They are commonly used in class design, reactive frameworks, DOM APIs, and computed property patterns.

**Beginner-Friendly Explanation:** Getters and setters are useful whenever you want property access to do more than just read or write a value.

### Common Use Cases

1. **Computed Properties**
   ```js
   class Circle {
     constructor(radius) { this.radius = radius; }
     get area() { return Math.PI * this.radius ** 2; }
   }
   ```

2. **Validation**
   ```js
   class User {
     set age(value) {
       if (value < 0) throw new RangeError("Age cannot be negative");
       this._age = value;
     }
     get age() { return this._age; }
   }
   ```

3. **Read-Only Properties**
   ```js
   const config = {
     _apiUrl: "https://api.example.com",
     get apiUrl() { return this._apiUrl; },
   };
   ```

4. **Lazy Loading**
   ```js
   class Data {
     get expensiveValue() {
       if (!this._cached) {
         this._cached = computeExpensiveValue();
       }
       return this._cached;
     }
   }
   ```

5. **Reactive Frameworks**
   ```js
   // Vue-style reactivity
   const vm = new Vue({
     data: { firstName: "Alice", lastName: "Smith" },
     computed: {
       fullName() { return `${this.firstName} ${this.lastName}`; },
     },
   });
   ```

6. **Encapsulation**
   ```js
   class BankAccount {
     #balance = 0;
     get balance() { return this.#balance; }
     deposit(amount) { this.#balance += amount; }
   }
   ```

7. **Chained Setters (Fluent APIs)**
   ```js
   const query = {
     _parts: [],
     set where(clause) { this._parts.push(`WHERE ${clause}`); },
     get sql() { return `SELECT * FROM t ${this._parts.join(" ")}`; },
   };
   query.where = "id = 1";
   console.log(query.sql); // "SELECT * FROM t WHERE id = 1"
   ```

### Annotated Code Examples

#### Example 1: Computed Property with Validation

```js
class Product {
  constructor(name, price) {
    this.name = name;
    this._price = 0;
    this.price = price; // Use setter for validation
  }

  get price() {
    return this._price;
  }

  set price(value) {
    if (value < 0) throw new RangeError("Price cannot be negative");
    this._price = value;
  }

  get formattedPrice() {
    return `$${this._price.toFixed(2)}`;
  }
}

const product = new Product("Widget", 19.99);
console.log(product.formattedPrice); // "$19.99"

try {
  product.price = -5;
} catch (err) {
  console.log(err.name); // "RangeError"
}
```

**Expected Output:**
```
$19.99
RangeError
```

**Why This Output:** The setter validates the price; the getter formats it.

#### Example 2: Lazy Computation

```js
class Fibonacci {
  constructor() {
    this._cache = new Map([[0, 0], [1, 1]]);
  }

  get(n) {
    return n; // Not a getter; a regular method for comparison
  }

  // Use a method instead of a getter for parameterised access.
  compute(n) {
    if (this._cache.has(n)) return this._cache.get(n);
    const result = this.compute(n - 1) + this.compute(n - 2);
    this._cache.set(n, result);
    return result;
  }
}

const fib = new Fibonacci();
console.log(fib.compute(10)); // 55
```

**Note:** Getters cannot take parameters; use methods for parameterised access.

### Real-World Cases

- **Vue:** Computed properties are getters.
- **MobX:** Observables use getters/setters.
- **DOM:** `element.innerHTML`, `element.value`, `element.classList`.
- **Node.js:** `process.env`, `stream.readable`.

---

## Performance

### Definitions

**Core Definition:** Performance of getters and setters refers to the runtime cost of using accessor properties compared to plain data properties.

**Technical Definition:** Getters and setters add a function call to each property access. Modern JavaScript engines (V8, SpiderMonkey) optimise accessor calls with inline caching, but accessors are still generally slower than direct property access. Performance impact depends on access frequency and engine optimisation.

**Beginner-Friendly Explanation:** Getters and setters are slower than plain properties because they run a function each time. For most code, the difference is negligible — but in tight loops or hot paths, it can matter.

### Performance Considerations

- **Function call overhead:** Each access invokes a function.
- **Optimisation:** Modern engines inline simple getters.
- **Hot paths:** Avoid accessors in performance-critical loops if possible.
- **Memory:** Accessor properties may use slightly more memory than data properties.
- **Reactivity:** Frameworks use accessors for reactivity, accepting the overhead.

### Annotated Code Examples

```js
const ITERATIONS = 10_000_000;

// Data property
const dataObj = { value: 42 };
let start = performance.now();
let sum = 0;
for (let i = 0; i < ITERATIONS; i++) sum += dataObj.value;
console.log(`Data property: ${(performance.now() - start).toFixed(2)} ms`);

// Accessor property
const accessorObj = {
  _value: 42,
  get value() { return this._value; },
};
start = performance.now();
sum = 0;
for (let i = 0; i < ITERATIONS; i++) sum += accessorObj.value;
console.log(`Accessor property: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
Data property: 8.50 ms
Accessor property: 15.20 ms
```

**Why This Output:** Accessors are typically ~2x slower than data properties due to function call overhead, though modern engines optimise this.

### Real-World Cases

- **Hot loops:** Prefer data properties or cache accessor results.
- **Reactive frameworks:** Accept accessor overhead for reactivity.
- **DOM:** Browser-optimised for common accessors.

---

## Pitfalls

### Definitions

**Core Definition:** Pitfalls of getters and setters are the common mistakes and surprising behaviours developers encounter when using accessors.

**Technical Definition:** Pitfalls include infinite recursion (getter/setter referencing itself), getter-only assignment failures, "TypeError: Cannot set property which has only a getter", confusion with `this`, and misuse with `Object.assign` (which triggers setters).

**Beginner-Friendly Explanation:** Getters and setters can be tricky. Here are the most common mistakes and how to avoid them.

### Common Pitfalls

#### 1. Infinite Recursion

```js
// BAD: Setter assigns to itself.
const obj = {
  set value(v) {
    this.value = v; // Infinite recursion!
  },
};

// GOOD: Use a different internal property.
const obj2 = {
  _value: 0,
  set value(v) { this._value = v; },
  get value() { return this._value; },
};
```

#### 2. Getter-Only Assignment Fails

```js
const obj = {
  get value() { return 42; },
};

obj.value = 100; // Silently fails (non-strict) or TypeError (strict)
console.log(obj.value); // 42
```

#### 3. `this` in Getters/Setters

```js
const obj = {
  value: 10,
  get doubled() { return this.value * 2; },
};

const { doubled } = obj; // Destructuring calls the getter with `this` = obj
console.log(doubled); // 20 (works because destructuring preserves this)
```

#### 4. Getters in `Object.assign`

```js
const source = {
  get value() { return 42; },
};
const target = Object.assign({}, source);
console.log(target.value); // 42 (getter value copied, not the getter)
```

#### 5. Non-Enumerable in Classes

```js
class MyClass {
  get value() { return 42; }
}

const obj = new MyClass();
console.log(Object.keys(obj)); // [] (getters are non-enumerable)
```

#### 6. `Object.freeze` and Accessors

```js
const obj = Object.freeze({
  get value() { return 42; },
});

// Object.freeze does not prevent the getter from being called,
// but it does prevent changing the property descriptor.
```

#### 7. Destructuring Triggers Getters

```js
const obj = {
  get expensive() {
    console.log("Computing...");
    return 42;
  },
};

const { expensive } = obj; // "Computing..." logged
```

### Annotated Code Examples

#### Example 1: Infinite Recursion Fix

```js
// BAD
const bad = {
  set value(v) {
    this.value = v; // Recursion
  },
};

// GOOD
const good = {
  _value: 0,
  set value(v) {
    this._value = v;
  },
  get value() {
    return this._value;
  },
};

good.value = 10;
console.log(good.value); // 10
```

**Expected Output:**
```
10
```

**Why This Output:** Using `_value` as internal storage avoids recursion.

#### Example 2: Getter-Only Assignment

```js
"use strict";

const obj = {
  get value() { return 42; },
};

try {
  obj.value = 100;
} catch (err) {
  console.log(err.name); // "TypeError"
  console.log(err.message); // "Cannot set property value of #<Object> which has only a getter"
}
```

**Expected Output:**
```
TypeError
Cannot set property value of #<Object> which has only a getter
```

**Why This Output:** In strict mode, assigning to a getter-only property throws.

### Real-World Cases

- **Vue:** Uses accessors carefully to avoid recursion.
- **Testing:** `Object.assign` tests must account for getters.
- **Code review:** Watching for getter-only assignment patterns.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Property Attributes — https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-property-attributes
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Method Definitions — https://tc39.es/ecma262/multipage/ecmascript-language-functions-and-classes.html#sec-method-definitions
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.defineProperty — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.defineproperty
- MDN Web Docs – get — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get
- MDN Web Docs – set — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set
- MDN Web Docs – Object.defineProperty() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
- MDN Web Docs – Property accessors — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_accessors
- MDN Web Docs – Working with objects — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects
- Vue.js Docs – Computed Properties — https://vuejs.org/guide/essentials/computed.html
- MobX Docs – Observables — https://mobx.js.org/observable-state.html
- Redux Docs – Immutable Update Patterns — https://redux.js.org/usage/structuring-reducers/immutable-update-patterns
- Exploring JS – Getters and setters — https://exploringjs.com/impatient-js/ch_objects.html#getters-and-setters
- V8 Blog – Fast Properties in V8 — https://v8.dev/blog/fast-properties
- You Don't Know JS – Getters & Setters — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/objects-classes/ch3.md