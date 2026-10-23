# JavaScript Inheritance Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Inheritance patterns are the recurring strategies JavaScript developers use to share behaviour and reuse code between objects, ranging from classical (class-based) inheritance to prototypal delegation, functional augmentation, and composition-based approaches.

**Technical Definition:** JavaScript implements inheritance through the prototype chain — an internal `[[Prototype]]` link that connects objects to other objects. Unlike class-based languages where a class is a separate template, JavaScript's classes are built on top of prototypal inheritance: the `class` syntax is largely a convenient abstraction over the same prototype-chain mechanism that has always existed. The principal inheritance patterns are: **classical** (emulating class hierarchies via constructors and `extends`), **prototypal** (creating objects that directly delegate to other objects via `Object.create` or `__proto__`), **functional** (using closures and factory functions to augment objects), **delegation** (OLOO — objects linked to other objects — where behaviour is looked up dynamically through the prototype chain rather than copied), and **composition** (assembling objects from smaller, independent behaviour units, often via mixins or delegation).

**Beginner-Friendly Explanation:** Inheritance is about how you share abilities between objects so you don't have to rewrite the same code over and over. JavaScript offers several ways to do this. You can use the familiar `class` and `extends` keywords (classical style), you can create objects that directly "link" to other objects (prototypal style), you can use factory functions and closures (functional style), or you can snap together small behaviour pieces like Lego bricks (composition). Each pattern has its place, and knowing the differences helps you choose the right tool for the job.

### Key Characteristics

- **One prototype link per object:** An object can delegate to only one other object at a time through its `[[Prototype]]`, though that object can itself have a prototype, forming a chain.
- **Classes are sugar:** The `class`/`extends` syntax does not introduce a new inheritance mechanism; it wraps prototypal inheritance in a cleaner interface.
- **Delegation is the engine:** Property lookup walks the prototype chain; this dynamic delegation is what makes inheritance work in JavaScript.
- **Composition is favoured:** Modern guidance strongly recommends "favour composition over inheritance" for flexible, loosely coupled designs.
- **Multiple patterns coexist:** Real-world codebases mix classical, prototypal, functional, and composition-based styles.

### Prerequisites

- Understanding of objects, functions, and the `new` operator.
- Familiarity with the prototype chain and `Object.create()`.
- Basic knowledge of closures (for functional inheritance).

### Related Programming Areas

- **Object-oriented programming:** Encapsulation, polymorphism, code reuse.
- **Design patterns:** Mixins, traits, decorators, strategy.
- **Functional programming:** Factory functions, closures, composition.

---

## Classical Inheritance

### Definitions

**Core Definition:** Classical inheritance is the class-based inheritance model familiar from languages like Java and C++, emulated in JavaScript using constructor functions and the prototype chain (or, in modern JavaScript, the `class` and `extends` keywords).

**Technical Definition:** In the classical pattern, a "class" is a constructor function whose `prototype` property holds shared methods. Instances are created with `new`, which sets the instance's `[[Prototype]]` to `Constructor.prototype`. Inheritance is achieved by setting the child constructor's prototype to an object whose `[[Prototype]]` is the parent's prototype (e.g., `Child.prototype = Object.create(Parent.prototype)`), and by calling the parent constructor inside the child constructor (e.g., `Parent.call(this, ...)`). Douglas Crockford described this emulation as "pseudo-classical" inheritance, noting that JavaScript is fundamentally class-free and uses prototypal inheritance instead. The ES6 `class` syntax provides a more ergonomic surface over the same mechanism.

**Beginner-Friendly Explanation:** Classical inheritance is the "traditional" style you might know from Java or C#: you define a parent class, then a child class that inherits from it. In JavaScript, you can write this with `class Parent` and `class Child extends Parent`. Under the hood, it still uses prototypes, but the syntax feels familiar.

### Purposes

- To model clear "is-a" hierarchies (a `Dog` is an `Animal`).
- To reuse code through a well-defined parent-child relationship.
- To provide a familiar mental model for developers coming from class-based languages.
- To leverage the modern `class` syntax for clean, declarative inheritance.

### Syntax Rules and Structure

**ES6 Class Syntax:**
```js
class Parent {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound.`; }
}

class Child extends Parent {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  speak() { return `${super.speak()} ${this.name} barks.`; }
}
```

**Pre-ES6 (Pseudo-Classical) Syntax:**
```js
function Parent(name) { this.name = name; }
Parent.prototype.speak = function () { return this.name + " makes a sound."; };

function Child(name, breed) {
  Parent.call(this, name);
  this.breed = breed;
}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
Child.prototype.speak = function () {
  return Parent.prototype.speak.call(this) + " " + this.name + " barks.";
};
```

**Component Breakdown:**
- `constructor`: Initialises instance properties.
- `extends` / prototype chain: Establishes the inheritance link.
- `super` / `Parent.call`: Invokes the parent constructor or method.

**Constraints and Limitations:**
- Single inheritance only (one `extends` clause).
- Deep hierarchies become brittle and hard to change.
- Private members are not truly private in the pre-ES6 pattern (closures or `#` fields are needed for encapsulation).

### Annotated Code Examples

#### Example 1: Classical Inheritance with `class`

```js
// Step 1: Define a parent class.
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound.`;
  }
}

// Step 2: Define a child class that extends the parent.
class Dog extends Animal {
  constructor(name, breed) {
    super(name); // call parent constructor
    this.breed = breed;
  }
  speak() {
    // Step 3: Override and extend the parent method using super.
    return `${super.speak()} ${this.name} barks.`;
  }
}

// Step 4: Instantiate and test.
const d = new Dog("Rex", "Labrador");
console.log(d.speak());        // "Rex makes a sound. Rex barks."
console.log(d instanceof Dog);   // true
console.log(d instanceof Animal); // true
```

**Expected Output:**
```
Rex makes a sound. Rex barks.
true
true
```

**Why This Output:** `Dog` extends `Animal`, so instances inherit `speak()` from `Animal.prototype`. `super(name)` initialises the parent part of the object, and `super.speak()` calls the parent method. `instanceof` works because the prototype chain is intact.

#### Example 2: Pseudo-Classical Inheritance (Pre-ES6)

```js
// Step 1: Parent constructor.
function Shape(color) {
  this.color = color;
}
Shape.prototype.describe = function () {
  return `A ${this.color} shape.`;
};

// Step 2: Child constructor.
function Circle(color, radius) {
  Shape.call(this, color); // call parent constructor
  this.radius = radius;
}
// Step 3: Set up the prototype chain.
Circle.prototype = Object.create(Shape.prototype);
Circle.prototype.constructor = Circle;
Circle.prototype.area = function () {
  return Math.PI * this.radius ** 2;
};

// Step 4: Test.
const c = new Circle("red", 5);
console.log(c.describe()); // "A red shape."
console.log(c.area().toFixed(2)); // "78.54"
```

**Expected Output:**
```
A red shape.
78.54
```

**Why This Output:** `Circle.prototype` is an object whose `[[Prototype]]` is `Shape.prototype`, so `describe()` is found via the chain. `Shape.call(this, color)` initialises the inherited `color` property.

### Real-World Cases

- **Framework base classes:** `class MyComponent extends React.Component`.
- **Error hierarchies:** `class ValidationError extends Error`.
- **Domain models:** `class PremiumUser extends User`.

---

## Prototypal Inheritance

### Definitions

**Core Definition:** Prototypal inheritance is JavaScript's native inheritance mechanism, where objects inherit directly from other objects through a prototype link, without requiring classes.

**Technical Definition:** Every object has an internal `[[Prototype]]` slot that references another object or `null`. When a property is accessed, the engine first checks the object itself, then its prototype, then the prototype's prototype, and so on, until the property is found or the chain ends at `null`. Objects can be created with a specific prototype using `Object.create(proto)`, and the prototype can be read or replaced with `Object.getPrototypeOf()` / `Object.setPrototypeOf()` (or the legacy `__proto__` accessor). The `class` syntax is a higher-level abstraction built on this same mechanism.

**Beginner-Friendly Explanation:** Instead of defining classes, you create an object and say "this new object should look up missing properties from that other object." If you ask a `rabbit` object for a property it doesn't have, JavaScript will check the `animal` object it's linked to. This direct object-to-object linking is the heart of prototypal inheritance.

### Purposes

- To create objects that share behaviour without class abstractions.
- To enable dynamic prototype chains that can be modified at runtime.
- To serve as the foundation on which classes are built.
- To support lightweight object creation with `Object.create()`.

### Syntax Rules and Structure

**Object.create():**
```js
const animal = { eats: true };
const rabbit = Object.create(animal);
rabbit.jumps = true;
console.log(rabbit.eats); // true (inherited)
```

**Constructor + Prototype:**
```js
function Animal(name) { this.name = name; }
Animal.prototype.eats = true;
const rabbit = new Animal("Rabbit");
```

**Constraints and Limitations:**
- An object can have only one `[[Prototype]]` at a time.
- Prototype chains can be mutated at runtime, which can be powerful but also dangerous.
- Property writes always affect the object itself, not the prototype (shadowing).

### Annotated Code Examples

#### Example 1: Direct Object-to-Object Inheritance

```js
// Step 1: Create a base object.
const animal = {
  eats: true,
  walk() {
    console.log(`${this.name} walks.`);
  },
};

// Step 2: Create an object linked to the base.
const rabbit = Object.create(animal);
rabbit.name = "Rabbit";
rabbit.jumps = true;

// Step 3: Access inherited and own properties.
console.log(rabbit.eats);   // true (inherited)
console.log(rabbit.jumps);  // true (own)
rabbit.walk();              // "Rabbit walks." (method inherited, this = rabbit)
```

**Expected Output:**
```
true
true
Rabbit walks.
```

**Why This Output:** `Object.create(animal)` sets `rabbit`'s `[[Prototype]]` to `animal`. Property lookup for `eats` and `walk` falls through to `animal`; `this` inside `walk` refers to `rabbit` because the method was called as `rabbit.walk()`.

#### Example 2: Constructor-Based Prototypal Inheritance

```js
// Step 1: Define a constructor and its prototype methods.
function Alligator(color) {
  this.color = color;
}
Alligator.prototype.introduce = function () {
  console.log("I am " + this.color);
};

// Step 2: Create a child constructor and link the prototypes.
function Croc(color) {
  Alligator.call(this, color);
}
Croc.prototype = Object.create(Alligator.prototype);

// Step 3: Instantiate and test.
const croc = new Croc("yellow");
croc.introduce(); // "I am yellow"
```

**Expected Output:**
```
I am yellow
```

**Why This Output:** `Croc.prototype` is an object whose prototype is `Alligator.prototype`. The `introduce` method is found via the chain.

### Real-World Cases

- **OLOO pattern:** Preferring objects linked to other objects over class hierarchies (Kyle Simpson's "You Don't Know JS").
- **Simple object factories:** `Object.create(null)` to create dictionary objects without inherited properties.
- **Polyfills:** Libraries often use `Object.create` for prototype setup.

---

## Functional Inheritance

### Definitions

**Core Definition:** Functional inheritance is a pattern where behaviour is shared by applying an augmenting function to an object instance, typically using closures to achieve private state.

**Technical Definition:** In the functional pattern, a factory function creates an object (often using an object literal or `Object.create`), adds properties and methods, and returns it. Inheritance is achieved by calling a "parent" factory function to create a base object, then extending it with additional properties before returning it. Closures over local variables provide true privacy, since those variables are not accessible outside the factory. Douglas Crockford popularised this approach in *JavaScript: The Good Parts*, favouring it over the pseudo-classical pattern for its simplicity and encapsulation.

**Beginner-Friendly Explanation:** Instead of classes or prototypes, you write a function that builds an object, gives it some methods, and returns it. If you want a "child" object, you call the parent builder first, then add more stuff. Variables inside the function stay private because they're trapped in a closure.

### Purposes

- To achieve genuine private state through closures.
- To avoid the complexity of `this` and prototype chains.
- To compose behaviour through function calls rather than inheritance hierarchies.
- To create objects with a simple, readable factory pattern.

### Syntax Rules and Structure

```js
function parentFactory(spec) {
  const that = {};            // fresh object
  that.name = spec.name;
  that.greet = function () {
    return `Hello, ${that.name}`;
  };
  return that;
}

function childFactory(spec) {
  const that = parentFactory(spec); // inherit by calling parent
  that.special = function () {
    return `${that.name} is special.`;
  };
  return that;
}
```

**Constraints and Limitations:**
- Each instance gets its own copy of every method (no sharing via prototype), which can be memory-intensive for large numbers of objects.
- No `instanceof` support (unless you manually set prototypes).
- `this` is avoided entirely; the returned object is captured in a variable (often named `that` or `self`).

### Annotated Code Examples

#### Example 1: Functional Inheritance with Closures

```js
// Step 1: Parent factory with a private variable.
function Animal(data) {
  const that = {};           // fresh object
  let energy = 100;          // private variable (closure)
  that.name = data.name;
  that.eat = function () {
    energy += 10;
    return `${that.name} eats. Energy: ${energy}`;
  };
  return that;
}

// Step 2: Child factory inherits by calling the parent factory.
function Cat(data) {
  const that = Animal(data); // inherit base behaviour
  that.sayHello = function () {
    return `Hello, I'm ${that.name}`;
  };
  return that;
}

// Step 3: Use the factory.
const myCat = Cat({ name: "Rufi" });
console.log(myCat.sayHello()); // "Hello, I'm Rufi"
console.log(myCat.eat());      // "Rufi eats. Energy: 110"
```

**Expected Output:**
```
Hello, I'm Rufi
Rufi eats. Energy: 110
```

**Why This Output:** `Cat` calls `Animal` to get a base object with the `eat` method and the private `energy` variable captured in its closure. `Cat` then adds `sayHello` before returning the extended object. The `energy` variable is inaccessible from outside, providing true privacy.

### Real-World Cases

- **Crockford-style modules:** The module pattern and functional inheritance are closely related.
- **Game entities (Boulder Bop):** Microsoft's HTML5 game skeleton uses functional inheritance for game objects with public and private methods.
- **Factory functions in modern code:** React hooks and similar patterns favour functions over classes.

---

## Delegation

### Definitions

**Core Definition:** Delegation is the mechanism by which an object, when asked for a property it does not own, forwards the lookup to another object (its prototype), which may forward it further up the chain.

**Technical Definition:** Delegation is the dynamic, runtime behaviour that powers prototypal inheritance. When a property is accessed on an object, the JavaScript engine performs a `[[Get]]` operation that walks the prototype chain until the property is found or `null` is reached. This is not copying; it is live lookup. The OLOO (Objects Linked to Other Objects) style, advocated by Kyle Simpson, embraces delegation directly: instead of modelling "classes" and "instances," you define behaviour objects and link other objects to them using `Object.create()`. Behaviour delegation is the conceptual foundation of JavaScript's object model.

**Beginner-Friendly Explanation:** Delegation means "if I can't do it myself, I'll ask my prototype." When you call a method on an object and that object doesn't have it, JavaScript automatically looks it up on the object's linked prototype. OLOO style makes this explicit: you link objects directly and let them delegate behaviour as needed.

### Purposes

- To enable dynamic, live property lookup without copying methods.
- To model behaviour sharing through object links rather than class hierarchies.
- To support OLOO style, which avoids the mental overhead of classes.
- To allow prototype chains to be inspected and modified at runtime.

### Syntax Rules and Structure

**OLOO with `Object.create`:**
```js
const Animal = {
  init(name) { this.name = name; return this; },
  speak() { return `${this.name} speaks.`; },
};

const dog = Object.create(Animal).init("Rex");
```

**Delegation with `Object.setPrototypeOf` (or `__proto__`):**
```js
const base = { greet() { return "Hello"; } };
const obj = {};
Object.setPrototypeOf(obj, base);
obj.greet(); // "Hello"
```

**Constraints and Limitations:**
- Delegation is read-only for lookup; writes always create or modify own properties.
- `Object.setPrototypeOf` is slow and should be avoided in hot code; prefer `Object.create`.
- Deep delegation chains can hurt performance.

### Annotated Code Examples

#### Example 1: OLOO-Style Delegation

```js
// Step 1: Define a behaviour object.
const Animal = {
  set name(value) { this._name = value; },
  get name() { return this._name; },
  speak() { return `${this.name} makes a sound.`; },
};

// Step 2: Link a new object to the behaviour object.
const dog = Object.create(Animal);
dog.name = "Rex";
dog.breed = "Lab";

// Step 3: Call the delegated method.
console.log(dog.speak()); // "Rex makes a sound."
console.log(Object.getPrototypeOf(dog) === Animal); // true
```

**Expected Output:**
```
Rex makes a sound.
true
```

**Why This Output:** `dog` does not own `speak`; the call delegates to `Animal` via the prototype link. `this` inside `speak` resolves to `dog`, so `this.name` reads the `name` property set on `dog`.

#### Example 2: Explicit Function Delegation (Composition via `call`)

```js
// Step 1: Define a behaviour function.
function canSwim() {
  return `${this.name} swims.`;
}

// Step 2: Use explicit delegation via call.
const fish = { name: "Nemo" };
console.log(canSwim.call(fish)); // "Nemo swims."
```

**Expected Output:**
```
Nemo swims.
```

**Why This Output:** `canSwim.call(fish)` explicitly delegates the function call to the `fish` object, binding `this` to it. This is composition through explicit function delegation, distinct from implicit prototype-chain delegation.

### Real-World Cases

- **OLOO libraries:** `stampit` supports delegation, concatenation, and functional inheritance.
- **Object behaviour sharing:** Linking objects to shared behaviour objects without classes.
- **Prototype-based frameworks:** Early JavaScript libraries used delegation extensively.

---

## Composition

### Definitions

**Core Definition:** Composition is a design approach where objects are built by combining smaller, independent behaviour units, rather than inheriting from a parent class.

**Technical Definition:** Composition models "has-a" or "can-do" relationships. An object "has" a logger, "has" a serializer, "can" emit events — it does not "is" a logger or "is" a serializer. In JavaScript, composition is implemented through techniques such as holding references to other objects and forwarding calls (delegation), mixing behaviour into objects or prototypes (`Object.assign`, subclass factories), or assembling objects from factory functions. The principle "favour composition over inheritance" is widely endorsed because composition avoids the tight coupling, fragile base-class problem, and rigid hierarchies that inheritance can create.

**Beginner-Friendly Explanation:** Composition is like building with Lego bricks instead of carving a statue from a single block. You create small, reusable pieces of behaviour and snap them together to form the object you need. If you need to change one behaviour, you swap that brick — you don't have to reshape the entire sculpture.

### Purposes

- To create flexible, loosely coupled designs.
- To avoid the diamond problem and deep inheritance hierarchies.
- To reuse behaviour across unrelated objects.
- To enable runtime behaviour swapping.
- To keep each component focused on a single responsibility.

### Syntax Rules and Structure

**Object composition (forwarding):**
```js
class Logger {
  log(msg) { console.log(msg); }
}
class Service {
  constructor() {
    this.logger = new Logger(); // has-a
  }
  doWork() { this.logger.log("Working"); }
}
```

**Mixin composition:**
```js
const canLog = {
  log(msg) { console.log(`[LOG] ${msg}`); },
};
Object.assign(Service.prototype, canLog);
```

**Subclass-factory composition:**
```js
const Timestamped = (Base) => class extends Base {
  constructor(...args) {
    super(...args);
    this.createdAt = new Date();
  }
};
class User extends Timestamped(BaseModel) {}
```

**Constraints and Limitations:**
- Can introduce more objects and indirection.
- Name collisions in mixin-based composition require discipline.
- Tracing where a method comes from may require tooling.

### Annotated Code Examples

#### Example 1: Composition via Forwarding ("has-a")

```js
// Step 1: Define a logger component.
class Logger {
  log(message) {
    console.log(`[LOG] ${message}`);
  }
}

// Step 2: Define a service that HAS a logger.
class UserService {
  constructor(logger) {
    this.logger = logger; // composition: has-a
  }
  createUser(name) {
    this.logger.log(`Creating user: ${name}`);
    return { name };
  }
}

// Step 3: Inject the dependency.
const service = new UserService(new Logger());
service.createUser("Alice");
```

**Expected Output:**
```
[LOG] Creating user: Alice
```

**Why This Output:** `UserService` does not inherit from `Logger`; it holds a reference to a `Logger` instance and forwards the logging call. The two classes are loosely coupled: `Logger` can be swapped for a different logger without changing `UserService`.

#### Example 2: Composition with Multiple Behaviours

```js
// Step 1: Define independent behaviour objects.
const canEat = {
  eat() { return `${this.name} eats.`; },
};
const canSleep = {
  sleep() { return `${this.name} sleeps.`; },
};
const canCode = {
  code() { return `${this.name} writes code.`; },
};

// Step 2: Compose a developer from behaviours.
function createDeveloper(name) {
  const dev = { name };
  return Object.assign(dev, canEat, canSleep, canCode);
}

// Step 3: Use the composed object.
const dev = createDeveloper("Alice");
console.log(dev.eat());  // "Alice eats."
console.log(dev.code()); // "Alice writes code."
```

**Expected Output:**
```
Alice eats.
Alice writes code.
```

**Why This Output:** `Object.assign` copies the behaviour methods onto the `dev` object. The developer is composed of three independent capabilities without any class hierarchy.

### Real-World Cases

- **React:** The React team recommends composition (props, children, hooks) over inheritance.
- **Vue 3 Composition API:** Replaces Vue 2 mixins with composable functions.
- **Dependency injection:** Services receive collaborators rather than extending them.
- **Game development:** Entity-component systems (ECS) are pure composition.

---

## When to Use Each

### Definitions

**Core Definition:** Choosing the right inheritance pattern depends on the relationship you are modelling, the flexibility you need, and the complexity you are willing to manage.

**Technical Definition:** The decision matrix centres on the **is-a vs. has-a** distinction: inheritance is appropriate for genuine, stable type hierarchies ("a Car is a Vehicle"), while composition is appropriate for capability sharing ("a Car has an Engine") and for situations where behaviour needs to be mixed, matched, or swapped. Prototypal delegation and functional inheritance offer lighter-weight alternatives when class hierarchies feel forced, and they excel in scenarios requiring dynamic linking or true privacy.

**Beginner-Friendly Explanation:** Use inheritance when the relationship is truly "this IS a special kind of that." Use composition when you want to give an object abilities without claiming it IS something else. When in doubt, composition is usually the safer, more flexible choice.

### Decision Guide

| Scenario | Recommended Pattern | Why |
|---|---|---|
| Clear, stable "is-a" hierarchy | Classical inheritance (`class extends`) | Natural fit; familiar to most developers. |
| Sharing behaviour across unrelated objects | Composition (mixins, forwarding) | No forced parent-child relationship. |
| Needing private state | Functional inheritance (closures) or private fields (`#`) | Closures or `#` fields provide true encapsulation. |
| Dynamic object linking without classes | Prototypal inheritance / OLOO | Direct object-to-object delegation. |
| Multiple behaviours from different sources | Composition / mixins | Single inheritance cannot express this cleanly. |
| Behaviour may need to change at runtime | Composition (dependency injection) | Components can be swapped. |
| Deeply nested, brittle hierarchy | Composition | Avoids fragile base-class problems. |

### Annotated Code Examples

#### Example 1: Choosing Inheritance vs. Composition

```js
// INHERITANCE: A Dog IS an Animal (is-a relationship).
class Animal { speak() { return "..."; } }
class Dog extends Animal { speak() { return "Woof"; } }

// COMPOSITION: A Car HAS an Engine (has-a relationship).
class Engine { start() { return "Vroom"; } }
class Car {
  constructor() {
    this.engine = new Engine(); // has-a
  }
  start() {
    return this.engine.start();
  }
}

console.log(new Dog().speak());  // "Woof" (inheritance)
console.log(new Car().start());  // "Vroom" (composition)
```

**Expected Output:**
```
Woof
Vroom
```

**Why This Output:** `Dog` inherits `speak` from `Animal` and overrides it — an "is-a" relationship. `Car` holds an `Engine` and forwards the `start` call — a "has-a" relationship. Each pattern matches the underlying domain relationship.

### Real-World Cases

- **True type hierarchies:** `class Admin extends User`, `class Circle extends Shape` — inheritance is natural.
- **Capability sharing:** Adding logging, serialization, or event support to multiple unrelated classes — composition is better.
- **Framework extension:** When a framework mandates `extends` (e.g., React class components), use inheritance; otherwise prefer composition.

---

## Inheritance vs. Composition

### Definitions

**Core Definition:** Inheritance vs. composition is the fundamental design trade-off between building objects through parent-child hierarchies (inheritance) and building them by combining independent parts (composition).

**Technical Definition:** Inheritance creates an **is-a** relationship and establishes a tight coupling between parent and child: changes to the parent propagate to all descendants, and the child is locked into the parent's taxonomy. Composition creates a **has-a** or **can-do** relationship and establishes loose coupling: components can be developed, tested, and replaced independently. The "fragile base class problem" and the "diamond problem" are classic inheritance pitfalls that composition avoids. Modern guidance — reflected in the design of languages like Rust and Go, and in the React and Vue ecosystems — strongly favours composition for most code-reuse scenarios.

**Beginner-Friendly Explanation:** Inheritance is like a family tree: you inherit traits from your parents, and if a grandparent changes something, it affects everyone below. Composition is like a toolbox: you pick and choose the tools you need, and swapping one tool doesn't break the others. Most modern developers prefer the toolbox.

### Comparison Table

| Aspect | Inheritance | Composition |
|---|---|---|
| **Relationship** | Is-a | Has-a / can-do |
| **Coupling** | Tight (child depends on parent) | Loose (components independent) |
| **Flexibility** | Rigid hierarchy | Flexible; swap components |
| **Multiple sources** | Limited (single parent) | Unlimited (many components) |
| **Fragile base class** | Yes — parent changes break children | No — components are isolated |
| **Diamond problem** | Possible in multiple inheritance | Avoided |
| **Code reuse** | Via base class | Via reusable components |
| **Best for** | Stable, well-understood type hierarchies | Evolving requirements, capability sharing |

### Key Principle

**"Favour composition over inheritance."** This does not mean inheritance is never appropriate — it means composition should be the default, and inheritance should be reserved for genuine, stable type hierarchies where the is-a relationship is unambiguous and unlikely to change.

### Annotated Code Examples

#### Example 1: The Same Problem Solved Two Ways

```js
// INHERITANCE APPROACH (brittle if behaviours vary independently)
class Animal { eat() { return "Eating"; } }
class FlyingAnimal extends Animal { fly() { return "Flying"; } }
class SwimmingAnimal extends Animal { swim() { return "Swimming"; } }

// A Duck needs BOTH fly and swim — inheritance forces duplication or awkward hierarchy.
class Duck extends FlyingAnimal {
  swim() { return "Swimming"; } // duplicated
}

// COMPOSITION APPROACH (flexible)
const canEat = { eat() { return "Eating"; } };
const canFly = { fly() { return "Flying"; } };
const canSwim = { swim() { return "Swimming"; } };

class Duck2 {
  constructor() {
    Object.assign(this, canEat, canFly, canSwim);
  }
}

const duck = new Duck2();
console.log(duck.fly());  // "Flying"
console.log(duck.swim()); // "Swimming"
```

**Expected Output:**
```
Flying
Swimming
```

**Why This Output:** The inheritance approach cannot cleanly give `Duck` both `fly` and `swim` without duplication, because it can extend only one parent. The composition approach assigns all three behaviour objects to the instance, giving it exactly the capabilities it needs.

### Real-World Cases

- **React:** "React has a powerful composition model, and we recommend using composition instead of inheritance to reuse code between components."
- **Vue 3:** The Composition API replaces mixins and inheritance for logic reuse.
- **Game engines:** Entity-component systems compose behaviours instead of building deep entity hierarchies.
- **Clean Code JavaScript:** "Prefer composition over inheritance" is a core principle.

---

## References

- MDN Web Docs – Inheritance and the prototype chain — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain
- MDN Web Docs – Object initializer — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer
- javascript.info – Prototypal inheritance — https://javascript.info/prototype-inheritance
- Douglas Crockford – Classical Inheritance in JavaScript — https://crockford.com/javascript/inheritance
- DigitalOcean – The JavaScript Prototypal Inheritance Pattern — https://www.digitalocean.com/community/tutorials/js-prototypal-pattern
- Educative – Understanding Prototypal Inheritance in JavaScript — https://www.educative.io/courses/rediscovering-javascript/lta/understanding-prototypal-inheritance
- jstips – What is Functional Inheritance? — https://raw.githubusercontent.com/loverajoel/jstips/refs/heads/master/_posts/en/javascript/2020-10-13-what-is-a-functional-inheritance.md
- Stack Overflow – Javascript pseudo-classical inheritance vs functional inheritance — https://stackoverflow.com/questions/8245635/javascript-pseudo-classical-inheritance-vs-functional-inheritance
- Stack Overflow – Concatenative inheritance and prototypal inheritance — https://stackoverflow.com
- Composition Over Inheritance (skill guide) — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/yanko-belov/code-craft/composition-over-inheritance/SKILL.md
- Learn JavaScript – Composition chapter — https://raw.githubusercontent.com/sumn2u/learn-javascript/7007c1bc4d16767e9a663cbc78aaf27e90473102/en/classes/composition.md
- Microsoft Web Dev for Beginners – Inheritance vs Composition — https://github.com/microsoft/Web-Dev-For-Beginners/tree/ca4f954c6769a697a2a40531a667abbc12715887/translations/ur/6-space-game/1-introduction
- You Don't Know JS – OLOO and Behaviour Delegation — https://books.google.com.sg/books?id=UY0lCgAAQBAJ
- Wikipedia – Composition over inheritance — https://en.wikipedia.org/wiki/Composition_over_inheritance