# JavaScript Prototypal Inheritance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Prototypal inheritance is JavaScript's native object-oriented model in which objects inherit properties and methods directly from other objects through a chain of prototype references.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §10.1), prototypal inheritance is implemented via the `[[Prototype]]` internal slot on every ordinary object. When a property is accessed and not found as an own property, the engine invokes the object's `[[Get]]` internal method, which delegates to the prototype's `[[Get]]`, continuing up the chain until the property is found or `null` is reached. The chain is established by `Object.create(proto)`, constructor functions with `prototype`, ES6 `class` with `extends`, or `Object.setPrototypeOf()`. The `instanceof` operator checks whether a constructor's `prototype` appears in an object's chain, and `isPrototypeOf()` performs the inverse check.

**Beginner-Friendly Explanation:** Prototypal inheritance is how JavaScript lets objects share behaviour. Instead of copying methods into every object, JavaScript links objects together in a "family tree" — when you ask an object for something it doesn't have, it asks its parent, then the grandparent, and so on. This is different from class-based languages like Java, where inheritance is defined by class hierarchies. In JavaScript, inheritance is between objects, not classes.

### Key Characteristics

- **Object-to-object inheritance:** Objects inherit directly from other objects.
- **Chain-based lookup:** Property access walks the `[[Prototype]]` chain.
- **Dynamic:** Chains can be modified at runtime (though this is discouraged).
- **Single-chain:** Each object has exactly one `[[Prototype]]`.
- **Delegation, not copying:** Inherited properties are not duplicated; they are delegated.
- **Rooted at `Object.prototype`:** Most chains terminate at `Object.prototype`, then `null`.
- **`class` is syntactic sugar:** ES6 classes desugar to prototype-based inheritance.

### Prerequisites

- Basic JavaScript syntax (objects, functions, properties).
- Understanding of prototypes and `[[Prototype]]`.
- Familiarity with constructor functions and `new`.
- Awareness of `this` binding.
- Basic knowledge of `Object.create()`.

### Related Programming Areas

- **Object-oriented programming:** Prototypal inheritance is JavaScript's OOP model.
- **ES6 classes:** Classes are syntactic sugar over prototypes.
- **Mixins:** Combining behaviours without a single chain.
- **Composition:** Preferring composition over inheritance.
- **Performance:** Deep chains have lookup overhead.
- **Framework design:** React, Vue, and libraries use prototypes.

---

## Prototypal Inheritance

### Definitions

**Core Definition:** Prototypal inheritance is the mechanism by which JavaScript objects inherit properties and methods directly from other objects through prototype chains.

**Technical Definition:** Per ECMA-262, when an object `O` is created with `Object.create(proto)`, `O.[[Prototype]]` is set to `proto`. Subsequent property accesses on `O` that are not own properties delegate to `proto.[[Get]]`, which may delegate further up the chain. This is distinct from classical inheritance, where classes are templates and instances are created from them; in prototypal inheritance, objects are created from other objects.

**Beginner-Friendly Explanation:** In prototypal inheritance, objects inherit from objects. You can create a new object based on an existing one, and it will automatically have access to all the original's properties and methods.

### Purposes

- To share behaviour between objects without duplicating code.
- To implement inheritance hierarchies.
- To enable polymorphism (different objects responding to the same method).
- To allow dynamic extension of objects at runtime.
- To support composition through delegation.

### Syntax Rules and Structure

**Via `Object.create()`:**
```js
const parent = { greet() { return "hi"; } };
const child = Object.create(parent);
child.greet(); // "hi"
```

**Via constructor functions:**
```js
function Parent() {}
Parent.prototype.greet = function() { return "hi"; };
function Child() {}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
new Child().greet(); // "hi"
```

**Via ES6 classes:**
```js
class Parent {
  greet() { return "hi"; }
}
class Child extends Parent {}
new Child().greet(); // "hi"
```

**Constraints and Limitations:**
- Each object has exactly one prototype (single inheritance).
- Deep chains have lookup overhead.
- Modifying prototypes affects all inheriting objects.
- `instanceof` works only with constructors that have a `prototype`.

### Annotated Code Examples

#### Example 1: Basic Prototypal Inheritance

```js
// Step 1: Define a prototype object.
const animal = {
  breathe() { return `${this.name} is breathing`; },
  describe() { return `${this.name} is a ${this.species}`; },
};

// Step 2: Create objects that inherit from it.
const dog = Object.create(animal);
dog.name = "Rex";
dog.species = "dog";

const cat = Object.create(animal);
cat.name = "Whiskers";
cat.species = "cat";

// Step 3: Use inherited methods.
console.log(dog.breathe());  // "Rex is breathing"
console.log(cat.describe()); // "Whiskers is a cat"
```

**Expected Output:**
```
Rex is breathing
Whiskers is a cat
```

**Why This Output:** `dog` and `cat` inherit `breathe` and `describe` from `animal` via the prototype chain.

#### Example 2: Multi-Level Inheritance

```js
// Step 1: Grandparent.
const livingThing = {
  isAlive() { return true; },
};

// Step 2: Parent inherits from grandparent.
const animal = Object.create(livingThing);
animal.breathe = function() { return `${this.name} breathes`; };

// Step 3: Child inherits from parent.
const dog = Object.create(animal);
dog.name = "Rex";
dog.bark = function() { return "Woof!"; };

// Step 4: Use all levels.
console.log(dog.isAlive());  // true (grandparent)
console.log(dog.breathe());  // "Rex breathes" (parent)
console.log(dog.bark());     // "Woof!" (own)
```

**Expected Output:**
```
true
Rex breathes
Woof!
```

**Why This Output:** `dog` inherits from `animal`, which inherits from `livingThing`. Property lookup walks the entire chain.

#### Example 3: Constructor-Based Inheritance

```js
// Step 1: Parent constructor.
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  return `${this.name} makes a sound.`;
};

// Step 2: Child constructor.
function Dog(name, breed) {
  Animal.call(this, name); // Call parent constructor
  this.breed = breed;
}

// Step 3: Set up prototype chain.
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

// Step 4: Child method.
Dog.prototype.bark = function() {
  return `${this.name} barks!`;
};

// Step 5: Create instances.
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

**Why This Output:** `Animal.call(this, name)` initialises inherited instance properties; `Object.create(Animal.prototype)` sets up the method inheritance; `instanceof` works for both constructors.

### Real-World Cases

- **DOM elements:** `HTMLButtonElement` inherits from `HTMLElement`, `Element`, `Node`, `EventTarget`.
- **Error subclasses:** `TypeError` inherits from `Error`.
- **Library hierarchies:** Backbone.js models, views.
- **ES6 classes:** Every `class` uses prototypal inheritance.

---

## Prototype Chain

### Definitions

**Core Definition:** The prototype chain is the sequence of objects linked by `[[Prototype]]` references, traversed during property lookup.

**Technical Definition:** Per ECMA-262, the prototype chain is traversed by the `[[Get]]` internal method. Starting from the receiver object, the engine checks each object's own property keys. If not found, it follows the `[[Prototype]]` reference to the next object. The chain terminates when a property is found or when `[[Prototype]]` is `null`. The `[[Set]]` method follows a more nuanced algorithm that considers writability and setters.

**Beginner-Friendly Explanation:** The prototype chain is JavaScript's "search path" for properties. When you access a property, JavaScript looks at the object, then its prototype, then its prototype's prototype, and so on until it finds the property or runs out of prototypes.

### Purposes

- To enable property and method inheritance.
- To provide predictable lookup order.
- To support `instanceof` and `isPrototypeOf`.
- To allow shared behaviour without duplication.

### Syntax Rules and Structure

**Chain structure:**
```
instance → Constructor.prototype → Parent.prototype → ... → Object.prototype → null
```

**Walking the chain:**
```js
let obj = instance;
while (obj !== null) {
  console.log(obj);
  obj = Object.getPrototypeOf(obj);
}
```

**Constraints and Limitations:**
- Deep chains have O(n) lookup cost.
- Modifying prototypes affects all inheriting objects.
- Property lookup stops at the first match.
- `null` terminates the chain.

### Annotated Code Examples

#### Example 1: Walking a Chain

```js
class Animal {
  breathe() { return "breathing"; }
}
class Dog extends Animal {
  bark() { return "woof"; }
}

const rex = new Dog();

// Walk the chain.
let current = rex;
const chain = [];
while (current !== null) {
  chain.push(current.constructor?.name || "null");
  current = Object.getPrototypeOf(current);
}
console.log(chain); // ["Dog", "Dog", "Animal", "Object"]
```

**Expected Output:**
```
[ 'Dog', 'Dog', 'Animal', 'Object' ]
```

**Why This Output:** `rex` → `Dog.prototype` → `Animal.prototype` → `Object.prototype` → `null`. (The first two are both "Dog" because `rex.constructor` is inherited from `Dog.prototype`.)

#### Example 2: Lookup Order

```js
const grandparent = { value: "grandparent" };
const parent = Object.create(grandparent);
parent.value = "parent";
const child = Object.create(parent);
child.value = "child";

console.log(child.value);  // "child" (own)
console.log(parent.value); // "parent" (own)
console.log(grandparent.value); // "grandparent" (own)

delete child.value;
console.log(child.value);  // "parent" (revealed)
delete parent.value;
console.log(child.value);  // "grandparent" (revealed)
```

**Expected Output:**
```
child
parent
grandparent
parent
grandparent
```

**Why This Output:** Each level shadows the one above. Deleting reveals the next.

### Real-World Cases

- **DOM:** `button` → `HTMLButtonElement.prototype` → `HTMLElement.prototype` → ... → `Object.prototype`.
- **Arrays:** `arr` → `Array.prototype` → `Object.prototype`.
- **Functions:** `fn` → `Function.prototype` → `Object.prototype`.
- **Classes:** Every `class` hierarchy.

---

## Inheritance Patterns

### Definitions

**Core Definition:** Inheritance patterns are the established techniques for setting up prototype chains in JavaScript.

**Technical Definition:** JavaScript supports several inheritance patterns: (1) **Prototype chain** via `Object.create()` or constructor `prototype`; (2) **Constructor stealing** via `Parent.call(this, ...)`; (3) **Combination inheritance** (prototype chain + constructor stealing); (4) **Prototypal inheritance** (objects inheriting from objects directly); (5) **Parasitic inheritance** (wrapping an object and enhancing it); (6) **Parasitic combination inheritance** (the most efficient pre-ES6 pattern); (7) **ES6 class inheritance** (syntactic sugar over prototypes).

**Beginner-Friendly Explanation:** There are several ways to set up inheritance in JavaScript. Some are older and more verbose; others are cleaner. ES6 classes are the modern preferred approach, but understanding the older patterns helps you read legacy code.

### Common Patterns

#### 1. Prototype Chain (Basic)

```js
function Parent() {}
Parent.prototype.greet = function() { return "hi"; };
function Child() {}
Child.prototype = new Parent(); // Inherits, but Parent runs
Child.prototype.constructor = Child;
```

**Pitfall:** Calling `new Parent()` runs the parent constructor, which may have side effects.

#### 2. Constructor Stealing

```js
function Parent(name) { this.name = name; }
function Child(name, age) {
  Parent.call(this, name); // "Steals" parent's initialisation
  this.age = age;
}
```

**Pitfall:** Does not inherit prototype methods.

#### 3. Combination Inheritance (Recommended Pre-ES6)

```js
function Parent(name) { this.name = name; }
Parent.prototype.greet = function() { return `Hi, ${this.name}`; };

function Child(name, age) {
  Parent.call(this, name); // Inherit instance properties
  this.age = age;
}
Child.prototype = Object.create(Parent.prototype); // Inherit methods
Child.prototype.constructor = Child;
```

**Benefits:** Inherits both instance properties and methods correctly.

#### 4. Prototypal Inheritance (Object-Based)

```js
const parent = { greet() { return "hi"; } };
const child = Object.create(parent);
child.name = "Alice";
```

**Benefits:** Simple, direct object-to-object inheritance.

#### 5. Parasitic Combination Inheritance (Most Efficient Pre-ES6)

```js
function inheritPrototype(child, parent) {
  const prototype = Object.create(parent.prototype);
  prototype.constructor = child;
  child.prototype = prototype;
}

function Parent(name) { this.name = name; }
Parent.prototype.greet = function() { return `Hi, ${this.name}`; };

function Child(name, age) {
  Parent.call(this, name);
  this.age = age;
}
inheritPrototype(Child, Parent); // Avoids calling Parent constructor
```

**Benefits:** Avoids calling the parent constructor twice.

#### 6. ES6 Class Inheritance (Modern)

```js
class Parent {
  constructor(name) { this.name = name; }
  greet() { return `Hi, ${this.name}`; }
}
class Child extends Parent {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
}
```

**Benefits:** Clean syntax, automatic prototype wiring.

### Annotated Code Examples

#### Example 1: Combination Inheritance

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  return `${this.name} makes a sound.`;
};

function Dog(name, breed) {
  Animal.call(this, name); // Constructor stealing
  this.breed = breed;
}
Dog.prototype = Object.create(Animal.prototype); // Prototype chain
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function() {
  return `${this.name} barks!`;
};

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

**Why This Output:** Combination inheritance inherits both instance properties and methods.

#### Example 2: ES6 Class Inheritance

```js
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound.`; }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  bark() { return `${this.name} barks!`; }
}

const rex = new Dog("Rex", "Labrador");
console.log(rex.speak()); // "Rex makes a sound."
console.log(rex.bark());  // "Rex barks!"
```

**Expected Output:**
```
Rex makes a sound.
Rex barks!
```

**Why This Output:** `extends` and `super()` automate the combination inheritance pattern.

### Real-World Cases

- **Framework base classes:** React `Component`, Vue `Vue`.
- **Error hierarchies:** `ValidationError extends Error`.
- **Domain models:** `Vehicle` → `Car` → `ElectricCar`.
- **DOM:** Built-in element hierarchies.

---

## `Object.create()`

### Definitions

**Core Definition:** `Object.create()` creates a new object with a specified prototype object.

**Technical Definition:** Per ECMA-262 §20.1.2.2, `Object.create(proto, propertiesObject)` creates a new ordinary object whose `[[Prototype]]` is `proto`. If `proto` is `null`, the object has no prototype. The optional second argument defines own properties via property descriptors.

**Beginner-Friendly Explanation:** `Object.create()` is the standard way to create an object that inherits from another object.

### Purposes

- To create objects with a specific prototype.
- To implement prototypal inheritance.
- To create objects without a prototype (`Object.create(null)`).
- To define properties with precise control.

### Syntax Rules and Structure

```js
Object.create(proto)
Object.create(proto, propertiesObject)
```

**Component Breakdown:**
- `proto`: The prototype object, or `null`.
- `propertiesObject` (optional): Property descriptors.

**Constraints and Limitations:**
- `proto` must be an object or `null`.
- For performance, prefer `Object.create()` over `Object.setPrototypeOf()`.
- `Object.create(null)` objects lack `Object.prototype` methods.

### Annotated Code Examples

```js
// Step 1: Basic inheritance.
const animal = { breathe() { return "breathing"; } };
const dog = Object.create(animal);
dog.name = "Rex";
console.log(dog.breathe()); // "breathing"

// Step 2: With property descriptors.
const person = Object.create(Object.prototype, {
  name: { value: "Alice", enumerable: true, writable: true },
  age: { value: 30, enumerable: true, writable: true },
});
console.log(person.name); // "Alice"

// Step 3: No prototype.
const pureMap = Object.create(null);
pureMap.key = "value";
console.log(pureMap.toString); // undefined
```

**Expected Output:**
```
breathing
Alice
undefined
```

**Why This Output:** `Object.create(animal)` sets `[[Prototype]]` to `animal`. `Object.create(null)` creates an object without inherited methods.

### Real-World Cases

- **Prototypal inheritance:** Standard pattern.
- **Pure dictionaries:** `Object.create(null)` for maps.
- **Library internals:** Many libraries use `Object.create` for inheritance.
- **Performance:** Preferred over `setPrototypeOf`.

---

## `Object.setPrototypeOf()`

### Definitions

**Core Definition:** `Object.setPrototypeOf()` sets the `[[Prototype]]` of an existing object to another object or `null`.

**Technical Definition:** Per ECMA-262 §20.1.2.22, `Object.setPrototypeOf(O, proto)` performs `O.[[SetPrototypeOf]](proto)`. If `O` is non-extensible and `proto` differs from its current prototype, it throws `TypeError`. In V8, this operation is deoptimising.

**Beginner-Friendly Explanation:** `Object.setPrototypeOf()` changes what an existing object inherits from. It's useful but slow.

### Purposes

- To change an object's prototype after creation.
- To polyfill `__proto__`.
- To implement dynamic inheritance.

### Syntax Rules and Structure

```js
Object.setPrototypeOf(obj, proto)
```

**Returns:** The object `obj`.

**Constraints and Limitations:**
- Very slow in V8 (deoptimises hidden class).
- Throws `TypeError` on non-extensible objects.
- Prefer `Object.create()` for new objects.

### Annotated Code Examples

```js
const proto = { greet() { return "hi"; } };
const obj = { name: "Alice" };

// Step 1: Set prototype.
Object.setPrototypeOf(obj, proto);
console.log(obj.greet()); // "hi"
console.log(Object.getPrototypeOf(obj) === proto); // true

// Step 2: Cannot set on non-extensible object.
const frozen = Object.freeze({});
try {
  Object.setPrototypeOf(frozen, proto);
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
hi
true
TypeError
```

**Why This Output:** Setting the prototype on a frozen object throws.

### Real-World Cases

- **Polyfills:** Implementing `__proto__`.
- **Dynamic inheritance:** Rare runtime prototype changes.
- **Avoid when possible:** Use `Object.create()` instead.

---

## `instanceof`

### Definitions

**Core Definition:** The `instanceof` operator checks whether an object's prototype chain includes the `prototype` property of a given constructor.

**Technical Definition:** Per ECMA-262 §13.10.2, `obj instanceof Constructor` calls `OrdinaryHasInstance(Constructor, obj)`, which checks whether `Constructor.prototype` appears in `obj`'s prototype chain by repeatedly following `[[Prototype]]`.

**Beginner-Friendly Explanation:** `instanceof` tells you if an object was created by a particular constructor (or inherits from one).

### Purposes

- To check the type of an object.
- To verify inheritance relationships.
- To implement polymorphic behaviour.

### Syntax Rules and Structure

```js
obj instanceof Constructor
```

**Returns:** Boolean.

**Constraints and Limitations:**
- Fails across realms (iframes).
- Can be fooled by modifying `Constructor.prototype`.
- Does not work with factory functions.
- `Symbol.hasInstance` can customise behaviour.

### Annotated Code Examples

```js
class Animal {}
class Dog extends Animal {}

const rex = new Dog();

console.log(rex instanceof Dog);    // true
console.log(rex instanceof Animal); // true
console.log(rex instanceof Object); // true
console.log([] instanceof Array);   // true
console.log([] instanceof Object);  // true
console.log("" instanceof String);  // false (primitives)
```

**Expected Output:**
```
true
true
true
true
true
false
```

**Why This Output:** `instanceof` walks the prototype chain. Primitives return `false` because they have no prototype chain.

### Real-World Cases

- **Type checking:** `err instanceof TypeError`.
- **Framework detection:** `obj instanceof Vue`.
- **DOM:** `el instanceof HTMLElement`.

---

## `isPrototypeOf()`

### Definitions

**Core Definition:** `Object.prototype.isPrototypeOf()` checks whether an object exists in the prototype chain of another object.

**Technical Definition:** Per ECMA-262 §20.1.3.3, `proto.isPrototypeOf(obj)` returns `true` if `proto` appears in `obj`'s prototype chain.

**Beginner-Friendly Explanation:** `isPrototypeOf()` is the inverse of `instanceof`: instead of asking "is this object an instance of that constructor?", you ask "is this object a prototype of that object?"

### Purposes

- To check inheritance relationships.
- To verify prototype chains.
- To implement `instanceof`-like checks without constructors.

### Syntax Rules and Structure

```js
proto.isPrototypeOf(obj)
```

**Returns:** Boolean.

### Annotated Code Examples

```js
const animal = { breathe() {} };
const dog = Object.create(animal);

console.log(animal.isPrototypeOf(dog));    // true
console.log(dog.isPrototypeOf(animal));    // false
console.log(Object.prototype.isPrototypeOf(dog)); // true

// Transitive.
const puppy = Object.create(dog);
console.log(animal.isPrototypeOf(puppy)); // true
```

**Expected Output:**
```
true
false
true
true
```

**Why This Output:** `isPrototypeOf` checks the entire chain, not just the immediate prototype.

### Real-World Cases

- **Inheritance checks:** Verifying prototype relationships.
- **Polyfills:** Implementing custom type checks.
- **Debugging:** Understanding prototype chains.

---

## Inheritance Performance

### Definitions

**Core Definition:** Inheritance performance refers to the runtime cost of property lookup through prototype chains and the memory implications of different inheritance patterns.

**Technical Definition:** Property lookup on the prototype chain is O(n) in the depth of the chain (though each step is a fast hash lookup). V8 optimises prototype chains with inline caches, but deep chains or frequently modified prototypes deoptimise the code. Memory usage is generally efficient because prototype methods are shared.

**Beginner-Friendly Explanation:** Inheritance is fast but not free. Deep chains are slower to look up. Modifying prototypes at runtime is very slow.

### Performance Considerations

- **Chain depth:** Shallower chains are faster.
- **Prototype modification:** `Object.setPrototypeOf` and `__proto__` deoptimise V8.
- **Inline caches:** Stable prototypes are optimised by V8.
- **Method sharing:** Prototype methods save memory.
- **Class syntax:** ES6 classes are optimised by engines.

### Annotated Code Examples

```js
// Deep chain (slower lookup).
const level0 = { value: 0 };
let current = level0;
for (let i = 1; i < 100; i++) {
  current = Object.create(current);
  current[`level${i}`] = i;
}
console.log(current.value); // 0 (walks 100 levels)

// Shallow chain (faster).
const shallowProto = { value: 0 };
const shallowChild = Object.create(shallowProto);
console.log(shallowChild.value); // 0 (walks 1 level)
```

**Note:** Both work correctly; the deep chain is slower.

### Real-World Cases

- **Avoid deep chains:** Keep inheritance shallow.
- **Prefer composition:** Use mixins over deep hierarchies.
- **Avoid `setPrototypeOf`:** Use `Object.create` at creation time.

---

## Inheritance Pitfalls

### Definitions

**Core Definition:** Inheritance pitfalls are the common mistakes and surprising behaviours developers encounter when using prototypal inheritance.

**Technical Definition:** Pitfalls arise from the dynamic nature of prototypes, the difference between `[[Prototype]]` and `prototype`, the `this` binding, shared mutable prototype state, and the performance cost of prototype modification.

**Beginner-Friendly Explanation:** Prototypal inheritance is powerful but has some traps. Knowing them helps you avoid bugs.

### Common Pitfalls

#### 1. Forgetting `Object.create()`

```js
// BAD: Inherits instance properties from a Parent instance.
function Parent(name) { this.name = name; }
function Child() {}
Child.prototype = new Parent(); // Calls Parent with no args

// GOOD: Inherit from Parent.prototype without calling Parent.
Child.prototype = Object.create(Parent.prototype);
```

#### 2. Forgetting to Restore `constructor`

```js
function Child() {}
Child.prototype = Object.create(Parent.prototype);
// Child.prototype.constructor is now Parent, not Child!

Child.prototype.constructor = Child; // Restore
```

#### 3. Shared Mutable State on Prototype

```js
function Counter() {}
Counter.prototype.items = []; // SHARED array!

const a = new Counter();
const b = new Counter();
a.items.push(1);
console.log(b.items); // [1] — surprise!
```

**Fix:** Initialise mutable state in the constructor.

#### 4. `this` Binding Issues

```js
function Person(name) { this.name = name; }
Person.prototype.greet = function() { return this.name; };

const alice = new Person("Alice");
const greet = alice.greet;
// greet(); // TypeError: Cannot read properties of undefined
```

**Fix:** Use `bind` or arrow functions in the constructor.

#### 5. Extending Built-ins

```js
class MyArray extends Array {}
const arr = new MyArray(1, 2, 3);
console.log(arr.map((x) => x * 2) instanceof MyArray); // true
// But some methods return MyArray, others return Array — inconsistent.
```

**Fix:** Be cautious when extending built-ins.

#### 6. `instanceof` Across Realms

```js
// In an iframe:
const iframeArray = iframe.contentWindow.Array;
// [] instanceof iframeArray === false (different Array constructor)
```

**Fix:** Use `Array.isArray()` or duck typing.

### Annotated Code Examples

#### Example 1: Shared Array Pitfall

```js
// BAD
function Team() {}
Team.prototype.members = [];

const t1 = new Team();
const t2 = new Team();
t1.members.push("Alice");
console.log(t2.members); // ["Alice"] — shared!

// GOOD
function Team2() {
  this.members = []; // Own array per instance
}
const t3 = new Team2();
const t4 = new Team2();
t3.members.push("Bob");
console.log(t4.members); // []
```

**Expected Output:**
```
[ 'Alice' ]
[]
```

**Why This Output:** Prototype properties are shared; instance properties are unique.

#### Example 2: Constructor Restoration

```js
function Parent() {}
function Child() {}

Child.prototype = Object.create(Parent.prototype);
console.log(Child.prototype.constructor === Child); // false
console.log(Child.prototype.constructor === Parent); // true

Child.prototype.constructor = Child;
console.log(Child.prototype.constructor === Child); // true
```

**Expected Output:**
```
false
true
true
```

**Why This Output:** Replacing `prototype` loses the original `constructor` link; restoring it fixes the issue.

### Real-World Cases

- **Code review:** Catching shared mutable state.
- **Library design:** Ensuring correct constructor links.
- **Framework migration:** Understanding pitfalls when moving from ES5 to ES6.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Ordinary Object Internal Methods — https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-ordinary-object-internal-methods-and-internal-slots
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – instanceof Operator — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-relational-operators
- MDN Web Docs – Inheritance and the prototype chain — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain
- MDN Web Docs – Object prototypes — https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes
- MDN Web Docs – Object.create() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create
- MDN Web Docs – Object.setPrototypeOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf
- MDN Web Docs – instanceof — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof
- MDN Web Docs – Object.prototype.isPrototypeOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isPrototypeOf
- MDN Web Docs – Classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- MDN Web Docs – Object.prototype.constructor — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor
- MDN Web Docs – Symbol.hasInstance — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/hasInstance
- Exploring JS – Prototype chains and classes — https://exploringjs.com/impatient-js/ch_proto-chains-classes.html
- You Don't Know JS – Objects & Classes — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/objects-classes/ch2.md
- Addy Osmani – Learning JavaScript Design Patterns — https://www.patterns.dev/
- V8 Blog – Fast Properties in V8 — https://v8.dev/blog/fast-properties
- V8 Blog – Setting Prototypes in V8 — https://v8.dev/blog/setting-prototypes
- Stoyan Stefanov – JavaScript Patterns — https://www.oreilly.com/library/view/javascript-patterns/9781449399115/