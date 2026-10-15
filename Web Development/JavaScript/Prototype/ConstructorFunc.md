# JavaScript Constructor Functions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Constructor functions are regular JavaScript functions designed to be called with the `new` operator to create and initialise new objects, forming the traditional foundation of JavaScript's object-oriented programming before ES6 classes.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §10.2.2), a constructor function is a function that supports the `[[Construct]]` internal method. When invoked with `new`, the engine creates a new ordinary object, sets its `[[Prototype]]` to the constructor's `prototype` property, binds `this` to the new object within the function body, executes the function, and returns the new object unless the function explicitly returns an object. Functions declared with `function` (and classes) have a `prototype` property that becomes the `[[Prototype]]` of instances. The `new.target` meta-property inside a constructor reveals whether the function was called with `new`.

**Beginner-Friendly Explanation:** A constructor function is a "blueprint" for creating objects. Instead of writing out the same object literal over and over, you define a function that describes what every object of that type should look like. Then you use `new` to stamp out as many copies as you need — each with its own data but sharing methods via the prototype.

### Key Characteristics

- **Called with `new`:** The `new` operator triggers the construction process.
- **`this` is the new object:** Inside the constructor, `this` refers to the object being created.
- **`prototype` property:** Every regular function has one; instances inherit from it.
- **Implicit return:** The new object is returned automatically unless the constructor returns an object.
- **`new.target`:** Detects whether the function was called with `new`.
- **Capitalisation convention:** Constructors conventionally start with a capital letter.
- **Superseded by `class`:** ES6 classes are syntactic sugar over constructor functions.

### Prerequisites

- Basic JavaScript syntax (functions, objects, properties).
- Understanding of `this` binding.
- Familiarity with prototypes and prototype chains.
- Awareness of the `new` operator's role.
- Basic knowledge of object literals.

### Related Programming Areas

- **Prototype-based inheritance:** Constructors rely on prototypes.
- **ES6 classes:** Classes desugar to constructor functions.
- **Factory functions:** An alternative pattern without `new`.
- **`instanceof` operator:** Uses the constructor's `prototype`.
- **`Object.create()`:** An alternative to `new`.
- **Legacy codebases:** Pre-ES6 code uses constructors extensively.

---

## Constructor Functions

### Definitions

**Core Definition:** A constructor function is a function intended to be invoked with `new` to create and initialise objects.

**Technical Definition:** Per ECMA-262 §10.2.2, when a function is called as a constructor via `[[Construct]]`, the following steps occur: (1) A new ordinary object is created. (2) The object's `[[Prototype]]` is set to the constructor's `prototype` property. (3) The function's `[[Call]]` is invoked with `this` bound to the new object. (4) If the function returns an object, that object is returned; otherwise, the new object is returned. Functions with `[[Construct]]` include ordinary functions, classes, and certain built-ins.

**Beginner-Friendly Explanation:** A constructor function is a function you call with `new` to create objects. It sets up the initial state of the object by assigning to `this`.

### Purposes

- To create multiple objects with the same structure.
- To define a "blueprint" for objects.
- To enable instance-specific data while sharing methods.
- To serve as the basis for ES6 classes.
- To support `instanceof` type checking.

### Syntax Rules and Structure

**General Syntax:**
```js
function ConstructorName(param1, param2) {
  // Instance properties
  this.param1 = param1;
  this.param2 = param2;
}

// Prototype methods (shared)
ConstructorName.prototype.method = function() { /* ... */ };

// Creating instances
const instance = new ConstructorName(arg1, arg2);
```

**Component Breakdown:**
- `ConstructorName`: Conventionally capitalised.
- `this.param1 = param1`: Instance property assignment.
- `ConstructorName.prototype.method`: Shared method.
- `new ConstructorName(...)`: Creates an instance.

**Syntax Rules:**
- Constructors should be called with `new`; calling without `new` may cause errors (in strict mode, `this` is `undefined`).
- Instance properties are assigned to `this`.
- Shared methods are defined on `prototype`.
- Constructors return `this` implicitly unless they return an object.
- Arrow functions cannot be constructors (no `[[Construct]]`).
- Methods (shorthand) cannot be constructors either.

**Constraints and Limitations:**
- Calling a constructor without `new` in non-strict mode pollutes the global scope.
- Instance properties defined in the constructor are duplicated per instance.
- Prototype methods are shared, saving memory.
- ES6 classes are preferred for new code.

### Annotated Code Examples

#### Example 1: Basic Constructor

```js
// Step 1: Define a constructor function.
function Person(name, age) {
  // Step 2: Assign instance properties.
  this.name = name;
  this.age = age;
}

// Step 3: Add a shared method to the prototype.
Person.prototype.greet = function() {
  return `Hi, I'm ${this.name} and I'm ${this.age}.`;
};

// Step 4: Create instances.
const alice = new Person("Alice", 30);
const bob = new Person("Bob", 25);

// Step 5: Use the instances.
console.log(alice.greet()); // "Hi, I'm Alice and I'm 30."
console.log(bob.greet());   // "Hi, I'm Bob and I'm 25."
```

**Expected Output:**
```
Hi, I'm Alice and I'm 30.
Hi, I'm Bob and I'm 25.
```

**Why This Output:** Each instance has its own `name` and `age`, but shares the `greet` method via the prototype.

#### Example 2: Instance vs Prototype Properties

```js
function Counter() {
  this.count = 0; // Instance property
}

Counter.prototype.increment = function() {
  this.count++;
  return this.count;
};

const c1 = new Counter();
const c2 = new Counter();

c1.increment();
c1.increment();
c2.increment();

console.log(c1.count); // 2
console.log(c2.count); // 1
console.log(Object.hasOwn(c1, "count"));       // true (own)
console.log(Object.hasOwn(c1, "increment"));   // false (inherited)
```

**Expected Output:**
```
2
1
true
false
```

**Why This Output:** `count` is an instance property (unique per instance); `increment` is a prototype method (shared).

#### Example 3: Returning an Object from a Constructor

```js
function Weird() {
  this.value = 1;
  return { custom: true }; // Overrides the implicit return
}

const obj = new Weird();
console.log(obj);          // { custom: true }
console.log(obj.value);    // undefined
```

**Expected Output:**
```
{ custom: true }
undefined
```

**Why This Output:** When a constructor returns an object, that object is used instead of `this`.

### Real-World Cases

- **Legacy code:** Pre-ES6 libraries and applications.
- **Built-in constructors:** `new Date()`, `new Map()`, `new Set()`.
- **Polyfills:** Emulating class behaviour in older environments.
- **Understanding ES6 classes:** Classes desugar to constructor functions.

---

## `new` Operator

### Definitions

**Core Definition:** The `new` operator creates an instance of a constructor function or class, setting up the prototype chain and binding `this` to the new object.

**Technical Definition:** Per ECMA-262 §13.3.5, the `NewExpression` production `new NewExpression` calls `Construct(ctor, [])`. The `Construct` abstract operation (§7.3.14) invokes the constructor's `[[Construct]]` internal method, which creates a new object, sets its `[[Prototype]]` to the constructor's `prototype` property, executes the constructor body with `this` bound to the new object, and returns the object (unless an object is explicitly returned).

**Beginner-Friendly Explanation:** The `new` operator is the "magic" that makes a constructor function produce a new object. It creates the object, connects it to the prototype, runs the constructor, and gives you the object back.

### Purposes

- To create instances from constructor functions.
- To set up the prototype chain automatically.
- To initialise new objects with `this`.
- To support `instanceof` checks.

### Syntax Rules and Structure

```js
new Constructor()
new Constructor(arg1, arg2)
new Constructor  // Parentheses optional
new new Constructor()  // New on the result of new
```

**Syntax Rules:**
- `new` binds tighter than function calls but looser than member access.
- `new Foo.bar()` parses as `new (Foo.bar)()`, not `(new Foo).bar()`.
- Parentheses can be omitted if no arguments: `new Date` is valid.
- `new` on a non-constructor throws `TypeError`.

**Constraints and Limitations:**
- Arrow functions and methods cannot be used with `new`.
- `new` on a built-in like `Math` throws `TypeError` (not a constructor).
- The constructor's `prototype` property must be an object.

### Annotated Code Examples

#### Example 1: Basic `new`

```js
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() {
  return `(${this.x}, ${this.y})`;
};

const p = new Point(3, 4);
console.log(p.toString());       // "(3, 4)"
console.log(p instanceof Point); // true
```

**Expected Output:**
```
(3, 4)
true
```

**Why This Output:** `new` creates an object with `Point.prototype` as its prototype, so `toString` is available and `instanceof` works.

#### Example 2: Parsing Precedence

```js
const obj = {
  Foo: function() { this.value = "Foo"; },
};

// new (obj.Foo)()
const a = new obj.Foo();
console.log(a.value); // "Foo"

// new (new Foo)() — nested
function Outer() {
  this.inner = new Inner();
}
function Inner() { this.value = "inner"; }

const outer = new Outer();
console.log(outer.inner.value); // "inner"
```

**Expected Output:**
```
Foo
inner
```

**Why This Output:** `new obj.Foo()` parses as `new (obj.Foo)()`, and nested `new` works as expected.

### Real-World Cases

- **Creating instances:** `new Date()`, `new Map()`, `new Set()`.
- **Legacy code:** Pre-ES6 object creation.
- **Framework internals:** Vue's reactivity uses `new` internally.
- **Understanding classes:** `new MyClass()` desugars to `new` on a constructor function.

---

## `new.target`

### Definitions

**Core Definition:** `new.target` is a meta-property available inside functions that reveals whether the function was called with `new` and, if so, which constructor was used.

**Technical Definition:** Per ECMA-262 §13.3.12, `new.target` is a `MetaProperty` available only inside function bodies (including class constructors and methods). Its value is the constructor being invoked via `new`, or `undefined` if the function was called without `new`. Inside a class constructor, `new.target` is the class being constructed (which may be a subclass).

**Beginner-Friendly Explanation:** `new.target` lets a function know whether it was called with `new`. If it was, `new.target` is the constructor; if not, it's `undefined`. This is useful for enforcing constructor usage.

### Purposes

- To enforce that a function is called with `new`.
- To detect the actual constructor being used (useful for inheritance).
- To write functions that can be called both with and without `new`.
- To implement abstract base classes.

### Syntax Rules and Structure

```js
function MyFunc() {
  if (!new.target) {
    throw new Error("MyFunc must be called with new");
  }
  this.value = 1;
}
```

**Syntax Rules:**
- Only available inside function bodies (not at top level, not in arrow functions).
- In arrow functions, `new.target` is inherited from the enclosing function.
- In a class constructor, `new.target` is the class being constructed.
- In a subclass constructor, `new.target` is the subclass.

**Constraints and Limitations:**
- Not available in modules' top level.
- Cannot be assigned to.
- Not available in arrow functions defined at the top level (throws SyntaxError if referenced).

### Annotated Code Examples

#### Example 1: Enforcing `new`

```js
function Person(name) {
  if (!new.target) {
    throw new Error("Person must be called with new");
  }
  this.name = name;
}

const alice = new Person("Alice");
console.log(alice.name); // "Alice"

try {
  Person("Bob"); // Forgot new
} catch (err) {
  console.log(err.message); // "Person must be called with new"
}
```

**Expected Output:**
```
Alice
Person must be called with new
```

**Why This Output:** `new.target` is `undefined` when called without `new`, triggering the error.

#### Example 2: Detecting Subclass Construction

```js
function Base() {
  console.log("Constructing:", new.target.name);
}

function Derived() {
  Base.call(this);
}

Object.setPrototypeOf(Derived.prototype, Base.prototype);

new Base();    // "Constructing: Base"
new Derived(); // "Constructing: Base" (Base.call doesn't set new.target)
```

**Expected Output:**
```
Constructing: Base
Constructing: Base
```

**Note:** With ES6 classes, `new.target` in the base class correctly reports the subclass when using `super()`.

#### Example 3: Dual-Purpose Function

```js
function createOrConstruct(value) {
  if (new.target) {
    this.value = value;
  } else {
    return { value };
  }
}

const a = new createOrConstruct(1);
const b = createOrConstruct(2);

console.log(a.value); // 1
console.log(b.value); // 2
```

**Expected Output:**
```
1
2
```

**Why This Output:** The function behaves differently based on whether `new` was used.

### Real-World Cases

- **Enforcing constructor usage:** APIs that require `new`.
- **Abstract classes:** Base classes that check `new.target`.
- **Mixins:** Detecting the actual class being constructed.
- **Framework internals:** Vue, Angular use `new.target` for class detection.

---

## `prototype` Property

### Definitions

**Core Definition:** The `prototype` property is a data property on constructor functions that becomes the `[[Prototype]]` of instances created with `new`.

**Technical Definition:** Per ECMA-262 §10.2.6, every function created by a `function` declaration or expression (except arrow functions, methods, and certain built-ins) has a `prototype` property. Its initial value is an ordinary object with a `constructor` property pointing back to the function. When `new` is used, the new object's `[[Prototype]]` is set to the constructor's `prototype` property.

**Beginner-Friendly Explanation:** The `prototype` property is the "shared storage" for a constructor. Methods and properties placed on it are inherited by all instances created with `new`.

### Purposes

- To share methods and properties across all instances.
- To save memory compared to defining methods in the constructor.
- To enable `instanceof` checks.
- To support prototype-based inheritance.

### Syntax Rules and Structure

```js
function Constructor() {}
Constructor.prototype = {
  constructor: Constructor, // Restore constructor
  method() { /* ... */ },
};
```

**Syntax Rules:**
- `prototype` is a property on functions, not on instances.
- The value can be replaced with a new object.
- Replacing `prototype` breaks the default `constructor` link unless restored.
- Instances created before replacement keep the old prototype.
- Arrow functions do not have a `prototype` property.

**Constraints and Limitations:**
- Replacing `prototype` affects only new instances.
- The `constructor` property is non-enumerable by default.
- Modifying `Object.prototype` affects all objects.

### Annotated Code Examples

#### Example 1: Shared Methods

```js
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  return `${this.name} makes a sound.`;
};

Animal.prototype.eat = function() {
  return `${this.name} is eating.`;
};

const dog = new Animal("Rex");
const cat = new Animal("Whiskers");

console.log(dog.speak()); // "Rex makes a sound."
console.log(cat.eat());   // "Whiskers is eating."

// Both share the same methods (memory-efficient).
console.log(dog.speak === cat.speak); // true
```

**Expected Output:**
```
Rex makes a sound.
Whiskers is eating.
true
```

**Why This Output:** Both instances inherit `speak` and `eat` from `Animal.prototype`, so they share the same function reference.

#### Example 2: Replacing the Prototype

```js
function Person(name) {
  this.name = name;
}

// BAD: Replacing prototype loses constructor link.
Person.prototype = {
  greet() { return `Hi, ${this.name}`; },
};

const alice = new Person("Alice");
console.log(alice.constructor === Person); // false (lost)
console.log(alice.constructor === Object); // true (inherited from Object.prototype)

// GOOD: Restore the constructor.
Person.prototype = {
  constructor: Person,
  greet() { return `Hi, ${this.name}`; },
};

const bob = new Person("Bob");
console.log(bob.constructor === Person); // true
```

**Expected Output:**
```
false
true
true
```

**Why This Output:** Replacing `prototype` with an object literal replaces the default `constructor` link. Restoring it manually fixes the issue.

### Real-World Cases

- **Class methods:** Every ES6 class method lives on `ClassName.prototype`.
- **Built-in prototypes:** `Array.prototype`, `String.prototype`.
- **Polyfills:** Adding methods to existing prototypes.
- **Legacy inheritance:** Pre-ES6 code uses `prototype` extensively.

---

## `constructor` Property

### Definitions

**Core Definition:** The `constructor` property is a reference on an object's prototype that points back to the constructor function that created the instance.

**Technical Definition:** Per ECMA-262 §10.2.6, when a function object is created, a `prototype` object is created with a `constructor` data property whose value is the function itself. The property is non-enumerable, writable, and configurable. Instances created with `new` inherit `constructor` from their prototype.

**Beginner-Friendly Explanation:** The `constructor` property is a "back link" from the prototype to the function that created it. It lets you check what function built an object.

### Purposes

- To identify the constructor of an instance.
- To create new instances from an existing instance's constructor.
- To restore the constructor link after replacing `prototype`.
- To support `instanceof`-like checks.

### Syntax Rules and Structure

```js
function Foo() {}
console.log(Foo.prototype.constructor === Foo); // true

const foo = new Foo();
console.log(foo.constructor === Foo); // true
```

**Syntax Rules:**
- `constructor` is inherited from the prototype.
- It is non-enumerable by default.
- Replacing `prototype` replaces `constructor` unless restored.
- Can be overridden by own properties.

**Constraints and Limitations:**
- Not a reliable type check (can be spoofed).
- Lost when `prototype` is replaced without restoration.
- `Object.create(null)` objects have no `constructor`.

### Annotated Code Examples

```js
function Foo() {}
const foo = new Foo();

// Step 1: Access constructor via instance.
console.log(foo.constructor === Foo); // true

// Step 2: Create a new instance from the constructor.
const bar = new foo.constructor();
console.log(bar instanceof Foo); // true

// Step 3: Own property shadows constructor.
foo.constructor = "fake";
console.log(foo.constructor); // "fake"
console.log(Foo.prototype.constructor === Foo); // true (unchanged)
```

**Expected Output:**
```
true
true
fake
true
```

**Why This Output:** `constructor` is inherited but can be shadowed by an own property.

### Real-World Cases

- **Factory functions:** `new obj.constructor()` to create similar objects.
- **Type checking:** Legacy alternative to `instanceof`.
- **Reflection:** Identifying the source of an object.

---

## Instance Properties

### Definitions

**Core Definition:** Instance properties are properties that belong to a specific object instance, defined in the constructor or added after creation.

**Technical Definition:** Per ECMA-262, instance properties are own properties of the instance object. They are created by `this.prop = value` in the constructor, by assignment on the instance, or via `Object.defineProperty`. They are not shared with other instances.

**Beginner-Friendly Explanation:** Instance properties are the "personal" data of an object — each instance has its own copy.

### Purposes

- To store unique state for each object.
- To customise instances.
- To hold per-object data that shouldn't be shared.

### Syntax Rules and Structure

```js
function Person(name, age) {
  this.name = name; // Instance property
  this.age = age;   // Instance property
}
```

**Syntax Rules:**
- Defined with `this.prop = value` in the constructor.
- Own properties of the instance.
- Not shared between instances.
- Can be added after creation: `instance.newProp = value`.

**Constraints and Limitations:**
- Functions defined as instance properties are duplicated per instance.
- Prefer prototype methods for shared behaviour.

### Annotated Code Examples

```js
function Person(name) {
  this.name = name;
  this.createdAt = new Date(); // Unique per instance
}

const alice = new Person("Alice");
const bob = new Person("Bob");

console.log(alice.name); // "Alice"
console.log(bob.name);  // "Bob"
console.log(alice.createdAt === bob.createdAt); // false (unique)

// Adding an instance property after creation
alice.email = "alice@example.com";
console.log(alice.email); // "alice@example.com"
console.log(bob.email);   // undefined (not shared)
```

**Expected Output:**
```
Alice
Bob
false
alice@example.com
undefined
```

**Why This Output:** Instance properties are unique per object.

### Real-World Cases

- **User data:** Name, email, ID.
- **State:** Counters, flags, caches.
- **Timestamps:** Creation time, last updated.

---

## Prototype Properties

### Definitions

**Core Definition:** Prototype properties are properties defined on a constructor's `prototype` object, shared by all instances via the prototype chain.

**Technical Definition:** Per ECMA-262, prototype properties are own properties of the constructor's `prototype` object. Instances inherit them through `[[Prototype]]`. They are shared by all instances and are typically methods.

**Beginner-Friendly Explanation:** Prototype properties are the "shared" parts of objects — methods and default values that all instances can use without duplicating memory.

### Purposes

- To share methods across instances (memory-efficient).
- To define default values that can be overridden.
- To enable inheritance and polymorphism.
- To support `instanceof`.

### Syntax Rules and Structure

```js
function Person(name) {
  this.name = name; // Instance property
}

Person.prototype.greet = function() { // Prototype property
  return `Hi, ${this.name}`;
};

Person.prototype.species = "human"; // Prototype property (default)
```

**Syntax Rules:**
- Defined after the constructor: `Constructor.prototype.prop = value`.
- Shared by all instances.
- Can be overridden by instance properties (shadowing).
- Accessible via `instance.prop`.

**Constraints and Limitations:**
- Modifying `Constructor.prototype` affects all instances.
- Prototype properties are not own properties of instances.
- Shared mutable objects on the prototype are a common pitfall.

### Annotated Code Examples

```js
function Person(name) {
  this.name = name;
}

// Prototype method (shared)
Person.prototype.greet = function() {
  return `Hi, ${this.name}`;
};

// Prototype default value
Person.prototype.role = "user";

const alice = new Person("Alice");
const bob = new Person("Bob");

console.log(alice.greet()); // "Hi, Alice"
console.log(bob.greet());   // "Hi, Bob"
console.log(alice.role);    // "user" (inherited)
console.log(bob.role);      // "user" (inherited)

// Override with instance property
alice.role = "admin";
console.log(alice.role); // "admin" (own)
console.log(bob.role);   // "user" (still inherited)
```

**Expected Output:**
```
Hi, Alice
Hi, Bob
user
user
admin
user
```

**Why This Output:** Prototype properties are shared until shadowed by an instance property.

### Real-World Cases

- **Class methods:** All ES6 class methods live on the prototype.
- **Built-in methods:** `Array.prototype.map`, `String.prototype.slice`.
- **Default values:** Shared configuration.
- **Polyfills:** Extending built-in prototypes.

---

## Constructor Inheritance

### Definitions

**Core Definition:** Constructor inheritance is the practice of making one constructor's instances inherit from another constructor's prototype, forming an inheritance hierarchy.

**Technical Definition:** Per ECMA-262, constructor inheritance in pre-ES6 JavaScript requires two steps: (1) Call the parent constructor with `Parent.call(this, ...)` inside the child constructor to initialise instance properties; (2) Set `Child.prototype = Object.create(Parent.prototype)` and restore `Child.prototype.constructor = Child` to inherit methods. ES6 classes automate this via `extends` and `super()`.

**Beginner-Friendly Explanation:** Constructor inheritance is like a family tree for constructors. A child constructor can inherit properties and methods from a parent constructor, and add its own.

### Purposes

- To share behaviour between related constructors.
- To create specialised versions of a base type.
- To enable polymorphism.
- To model real-world "is-a" relationships.

### Syntax Rules and Structure

**Pre-ES6 (function-based):**
```js
function Parent(name) {
  this.name = name;
}
Parent.prototype.greet = function() { return `Hi, ${this.name}`; };

function Child(name, age) {
  Parent.call(this, name); // Call parent constructor
  this.age = age;
}
Child.prototype = Object.create(Parent.prototype); // Inherit methods
Child.prototype.constructor = Child;               // Restore constructor
```

**ES6 (class-based):**
```js
class Parent {
  constructor(name) { this.name = name; }
  greet() { return `Hi, ${this.name}`; }
}
class Child extends Parent {
  constructor(name, age) {
    super(name); // Call parent constructor
    this.age = age;
  }
}
```

**Constraints and Limitations:**
- Pre-ES6 inheritance requires manual prototype wiring.
- `Parent.call(this, ...)` does not set up the prototype chain.
- Forgetting to restore `constructor` breaks `instanceof`-like checks.
- ES6 classes are preferred for new code.

### Annotated Code Examples

#### Example 1: Pre-ES6 Inheritance

```js
// Step 1: Define the parent constructor.
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  return `${this.name} makes a sound.`;
};

// Step 2: Define the child constructor.
function Dog(name, breed) {
  Animal.call(this, name); // Call parent constructor
  this.breed = breed;
}

// Step 3: Set up prototype chain.
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

// Step 4: Add child-specific method.
Dog.prototype.bark = function() {
  return `${this.name} barks!`;
};

// Step 5: Create instances.
const rex = new Dog("Rex", "Labrador");
console.log(rex.speak()); // "Rex makes a sound." (inherited)
console.log(rex.bark());  // "Rex barks!" (own)
console.log(rex instanceof Dog);    // true
console.log(rex instanceof Animal); // true
```

**Expected Output:**
```
Rex makes a sound.
Rex barks!
true
true
```

**Why This Output:** `Animal.call(this, name)` initialises the name; `Object.create(Animal.prototype)` sets up inheritance; `instanceof` works for both.

#### Example 2: ES6 Class Inheritance

```js
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound.`; }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Call parent constructor
    this.breed = breed;
  }
  bark() { return `${this.name} barks!`; }
}

const rex = new Dog("Rex", "Labrador");
console.log(rex.speak()); // "Rex makes a sound."
console.log(rex.bark());  // "Rex barks!"
console.log(rex instanceof Dog);    // true
console.log(rex instanceof Animal); // true
```

**Expected Output:**
```
Rex makes a sound.
Rex barks!
true
true
```

**Why This Output:** ES6 classes automate prototype wiring; `super(name)` calls the parent constructor.

### Real-World Cases

- **Framework base classes:** React's `Component` class.
- **Library hierarchies:** Error subclasses.
- **Domain models:** `Vehicle` → `Car` → `ElectricCar`.
- **DOM element classes:** `HTMLElement` → `HTMLButtonElement`.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Constructors — https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-ecmascript-function-objects-construct-argumentslist-newtarget
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – The new Operator — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-new-operator
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – new.target — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-meta-properties
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Function.prototype — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-function.prototype
- MDN Web Docs – Object.prototype.constructor — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor
- MDN Web Docs – new operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new
- MDN Web Docs – new.target — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target
- MDN Web Docs – Function.prototype — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype
- MDN Web Docs – Working with objects — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects
- MDN Web Docs – Object prototypes — https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes
- MDN Web Docs – Inheritance and the prototype chain — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain
- MDN Web Docs – Classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- MDN Web Docs – instanceof — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof
- You Don't Know JS – Objects & Classes — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/objects-classes/ch4.md
- Exploring JS – Classes — https://exploringjs.com/impatient-js/ch_classes.html
- Stoyan Stefanov – JavaScript Patterns — https://www.oreilly.com/library/view/javascript-patterns/9781449399115/
- Addy Osmani – Learning JavaScript Design Patterns — https://www.patterns.dev/vanilla/