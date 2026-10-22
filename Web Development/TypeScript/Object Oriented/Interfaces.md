# Interfaces in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A TypeScript **interface** is a purely **type-level construct** that names an object's shape by specifying the properties and methods that values of that type must possess. Interfaces are **erased** during compilation and have no runtime representation; they exist exclusively to enable structural type checking, contract documentation, and tooling intelligence. A class **implements** an interface to declare—and have the compiler verify—that its instances conform to the interface's contract.

**Technical Definition.** An interface declaration introduces a named **object type** into the type space. It may contain property signatures (required, optional, or `readonly`), method signatures, call signatures, construct signatures, and index signatures. Interfaces support **declaration merging**: multiple declarations with the same name in the same scope merge into a single type. A class's `implements` clause is a **compile-time assertion**: the compiler verifies that the class's public instance shape is assignable to each named interface, but the clause emits no JavaScript code. Unlike `extends`, `implements` establishes **no prototype-chain relationship** and provides **no inherited implementation**.

**Beginner-Friendly Explanation.** An interface is a **contract that describes what an object must look like**. It lists the properties (data) and methods (actions) an object must have, without providing any actual values or code. When a class says `implements SomeInterface`, it is **promising** to provide everything that interface requires. TypeScript checks that promise at compile time. If the class is missing a required member or has the wrong type, you get an error before the code ever runs. Interfaces are erased completely when TypeScript becomes JavaScript—they are a development-time safety net, not a runtime enforcement mechanism.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Pure type construct** | Interfaces exist only at compile time and are entirely erased in emitted JavaScript. |
| **Structural contract** | An interface describes a shape; any value with that shape is compatible, regardless of nominal identity. |
| **Implementation assertion** | `implements` is a compile-time check; it does not create runtime inheritance or prototype linkage. |
| **Multiple implementation** | A single class may implement any number of interfaces, separated by commas. |
| **Open and mergeable** | Interfaces with the same name in the same scope automatically merge (declaration merging). |
| **No implementation inheritance** | `implements` provides no method bodies; the class must supply every required member. |
| **Optional and readonly members** | Interfaces support `?` for optional members and `readonly` for immutable members. |
| **Extensible** | Interfaces may extend one or more other interfaces, composing their contracts. |

---

### Prerequisites

- **TypeScript basics**: type annotations, type aliases, and structural typing.
- **JavaScript classes**: `class`, methods, properties, and instantiation.
- **Object-oriented concepts**: contracts, polymorphism, and the distinction between “is-a” (`extends`) and “can-do” (`implements`).
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Structural typing** and **duck typing** (TypeScript’s compatibility model).
- **Class inheritance** (`extends`) — contrast with interface implementation.
- **Abstract classes** — a hybrid of contract and partial implementation.
- **Dependency inversion** and **interface segregation** (SOLID principles).
- **Declaration merging** for augmenting third-party library types.

---

### Core Concepts / Features

The following sections cover the requested concepts. Each follows the uniform structure.

---

## 1. Interfaces

### Definitions

**Core Definition.** An **interface** is a named type declaration that specifies the **shape** — the properties, methods, and signatures — that a value must provide. It contains no implementation and exists solely for compile-time checking and documentation.

**Technical Definition.** An interface declaration `interface Name { ... }` introduces a type into the type space. It may include property signatures (`prop: Type`), optional properties (`prop?: Type`), `readonly` properties (`readonly prop: Type`), method signatures (`method(params): ReturnType`), call signatures (`(args): ReturnType`), construct signatures (`new (args): Type`), and index signatures (`[key: string]: Type`). Interfaces are **open**: multiple declarations with the same name in the same scope merge. Interfaces can extend other interfaces via `extends`. They are **erased** during compilation and have no runtime counterpart.

**Beginner-Friendly Explanation.** An interface is **a checklist for objects**. It says “if you want to be a `User`, you must have a `name` that is a string and an `email` that is a string.” Any object that has those things is a `User` as far as TypeScript is concerned—even if it was never explicitly declared to be one. Interfaces are only for the type checker; they vanish when the code runs.

### Purposes

- **To name and document object shapes** so the same contract can be reused across a codebase.
- **To define contracts for classes** that must be verified at compile time.
- **To enable structural polymorphism** where any conforming value is accepted, regardless of its origin.
- **To support declaration merging** for augmenting library types and environment declarations.
- **To serve as the preferred tool for public object-type APIs** because interfaces produce clearer compiler errors and support open extension.

### Syntax Rules and Structure

**General syntax (interface declaration):**

```
interface InterfaceName {
  propertyName: Type;
  optionalProperty?: Type;
  readonly immutableProperty: Type;
  methodName(params: Type): ReturnType;
  [index: string]: Type;
}
```

**Component breakdown:**

- `interface` — declaration keyword.
- `InterfaceName` — identifier (PascalCase convention).
- `propertyName: Type` — required property and its type.
- `optionalProperty?: Type` — optional property; its type is `Type | undefined`.
- `readonly immutableProperty: Type` — property that cannot be reassigned after creation.
- `methodName(params): ReturnType` — method signature.
- `[index: string]: Type` — index signature for dynamic keys.

**Syntax rules.**

- Interfaces may extend one or more interfaces: `interface Derived extends Base1, Base2 { ... }`.
- Interfaces may be generic: `interface Box<T> { value: T; }`.
- Interfaces are **open**: redeclaring the same interface name in the same scope merges the declarations.
- Interfaces may describe function types via call signatures: `interface Fn { (x: number): string; }`.
- Interfaces may describe constructor types via construct signatures: `interface Ctor { new (): object; }`.

**Constraints and limitations.**

- Interfaces **cannot** represent unions, intersections, primitives, tuples, or mapped/conditional types directly; use `type` aliases for those.
- Interfaces are erased and cannot be used with `instanceof`.
- Declaration merging can be surprising; conflicting property types across merged declarations produce errors.
- Interfaces do not exist at runtime; runtime validation requires user-defined type guards.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic interface describing an object shape

```typescript
// example_interface_basic.ts
// Step 1: Declare an interface.
interface User {
  name: string;
  age: number;
  email?: string;         // Optional
  readonly id: string;    // Immutable after creation
}

// Step 2: Create a conforming object.
const user: User = {
  name: "Ada",
  age: 36,
  id: "u1",
};

// Step 3: Access properties.
console.log(user.name); // Expected: Ada
console.log(user.age);  // Expected: 36
console.log(user.email); // Expected: undefined

// user.id = "u2"; // Error: Cannot assign to 'id' because it is a read-only property.
```

**Setup.** Compile with `npx tsc example_interface_basic.ts --strict`.

**Expected Output:**
```
Ada
36
undefined
```

**Why this output occurs.** The object literal is checked against `User`: `name` and `age` are required and present; `email` is optional and omitted; `id` is readonly and initialized once. The compiler enforces these constraints; at runtime, the interface is erased and only the object properties remain.

---

#### Example 2: Interface with method and index signatures

```typescript
// example_interface_methods.ts
interface StringMap {
  [key: string]: string | undefined;
}

interface Greeter {
  greet(name: string): string;
  defaultGreeting: string;
}

const map: StringMap = { hello: "world", foo: "bar" };
console.log(map["hello"]); // Expected: world

const greeter: Greeter = {
  defaultGreeting: "Hello",
  greet(name) {
    return `${this.defaultGreeting}, ${name}`;
  },
};

console.log(greeter.greet("Ada")); // Expected: Hello, Ada
```

**Expected Output:**
```
world
Hello, Ada
```

**Why this output occurs.** `StringMap` uses an index signature to allow arbitrary string keys with string-or-undefined values. `Greeter` requires a `greet` method and a `defaultGreeting` property. The object literal satisfies both. At runtime, the methods execute normally; the interface annotations are erased.

---

### Real-World Cases

- **API response contracts**: `interface ApiResponse<T> { data: T; status: number; error?: string; }`.
- **Configuration shapes**: `interface AppConfig { port: number; host: string; }`.
- **React props**: `interface ButtonProps { label: string; onClick: () => void; }`.
- **Library type augmentation**: merging a `Window` interface to add custom global properties.

---

## 2. Implementing Interfaces

### Definitions

**Core Definition.** **Implementing an interface** is the act by which a class declares—via the `implements` clause—that its instances conform to the interface’s contract. The compiler verifies that the class provides every required member with a compatible type; if any member is missing or mistyped, compilation fails.

**Technical Definition.** The `implements` clause on a class is a **compile-time assertion** that the class’s public instance type is assignable to each named interface. It establishes **no runtime relationship**: the emitted JavaScript contains no reference to the interface, and no prototype linkage is created between the class and the interface (interfaces have no runtime prototype). The check is structural: the class must have **at least** the members required by the interface, with compatible types. Additional members are permitted. If the class fails to implement a required member, the compiler reports an error naming the missing or incompatible member.

**Beginner-Friendly Explanation.** Implementing an interface means **a class promises to follow the interface’s checklist**. If the interface says “you must have a `speak()` method,” the class must provide one. TypeScript checks the class against the checklist. The interface itself does not give the class any code—it only demands that the class supply the code. Once compiled, the interface disappears, but the class’s methods remain.

### Purposes

- **To enforce class contracts at compile time** so missing or mistyped members are caught before runtime.
- **To decouple classes from specific implementations**, enabling polymorphism through the interface type.
- **To document the intended public shape** of a class without exposing implementation details.
- **To enable dependency inversion**: high-level code can depend on interfaces rather than concrete classes.
- **To combine multiple behavioral contracts** in a single class without multiple inheritance.

### Syntax Rules and Structure

**General syntax (class implementing an interface):**

```
class ClassName implements InterfaceName {
  // must provide all required members
}
```

**Component breakdown:**

- `class ClassName` — the implementing class.
- `implements InterfaceName` — the contract assertion.
- `{ ... }` — class body; every required interface member must be present with a compatible type.

**Syntax rules.**

- A class may implement **one or more** interfaces: `implements A, B, C`.
- Every **required** member of each interface must be implemented.
- Optional interface members do not need to be implemented.
- Implemented members must be at least as accessible as the interface member (typically `public`).
- The class may have **additional** members beyond those required by the interface.
- A class may both `extends` a base class and `implements` interfaces: `class C extends Base implements A, B`.

**Constraints and limitations.**

- `implements` is **compile-time only**; it emits no runtime code and creates no prototype relationship.
- A class cannot implement an interface that requires a **private** member unless the class itself declares a compatible private member (nominal-like behavior).
- The interface’s members must be satisfiable by the class’s **public instance shape**; static members do not satisfy instance-member requirements.
- Implementing an interface does **not** inherit any implementation; the class must write every method body.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic class implementing an interface

```typescript
// example_implements_basic.ts
// Step 1: Define the interface.
interface Alarm {
  alert(): void;
}

// Step 2: Implement it in a class.
class SecurityDoor implements Alarm {
  alert(): void {
    console.log("SecurityDoor alert");
  }
}

// Step 3: Use the class.
const door = new SecurityDoor();
door.alert(); // Expected: SecurityDoor alert
```

**Expected Output:**
```
SecurityDoor alert
```

**Why this output occurs.** `SecurityDoor` declares `implements Alarm` and provides an `alert` method. The compiler verifies that the method signature matches. At runtime, the interface is erased; only the class and its method remain. If `alert` were missing, compilation would fail with an error.

---

#### Example 2: Implementing multiple members and optional members

```typescript
// example_implements_members.ts
interface Vehicle {
  speed: number;
  accelerate(amount: number): void;
  readonly vin: string;
  owner?: string; // Optional — does not need to be implemented.
}

class Car implements Vehicle {
  speed = 0;
  readonly vin = "1HGCM82633A004352";

  accelerate(amount: number): void {
    this.speed += amount;
  }
}

const car = new Car();
car.accelerate(50);
console.log(car.speed); // Expected: 50
console.log(car.vin);   // Expected: 1HGCM82633A004352
```

**Expected Output:**
```
50
1HGCM82633A004352
```

**Why this output occurs.** `Car` implements `speed`, `accelerate`, and `vin`. The optional `owner` is omitted, which is legal. `readonly vin` is a class field that satisfies the interface’s readonly requirement. The compiler checks all required members; the optional member is not enforced.

---

#### Example 3: Interface implementation is structural, not nominal

```typescript
// example_implements_structural.ts
interface Named {
  name: string;
}

class Person {
  constructor(public name: string) {}
}

// Person does NOT declare `implements Named`, but it is structurally compatible.
const named: Named = new Person("Ada");
console.log(named.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `Person` has a public `name: string` property, so its instance type is structurally assignable to `Named`. The `implements` keyword is not required for assignability; it is only a **declaration of intent** and a **compiler assertion**. This demonstrates that TypeScript’s interface implementation checks are built on structural compatibility.

---

### Real-World Cases

- **Repository classes**: `class UserRepository implements IRepository<User> { ... }`.
- **Service implementations**: `class EmailService implements INotificationService { ... }`.
- **Mock objects in tests**: a mock class implements the same interface as the real service, enabling dependency injection.
- **Framework contracts**: `class MyComponent implements OnInit, OnDestroy { ... }` in Angular.

---

## 3. `implements`

### Definitions

**Core Definition.** The **`implements` keyword** is the syntactic operator that attaches an interface contract to a class declaration. It tells the compiler: “verify that this class conforms to the named interface(s).”

**Technical Definition.** In TypeScript’s grammar, the `implements` clause appears after the class name (and optional `extends` clause): `class C extends Base implements A, B { ... }`. The clause accepts a comma-separated list of **type expressions** that resolve to interfaces, class types, or object types. The compiler checks that the class’s **public instance type** is assignable to each listed type. The `implements` clause is **erased**; it produces no JavaScript. It is purely a static assertion.

**Beginner-Friendly Explanation.** `implements` is **the word that creates the promise**. Writing `class Dog implements Animal` means “I promise that `Dog` follows the `Animal` contract.” TypeScript checks the promise. If `Dog` is missing something `Animal` requires, you get an error. The word itself does nothing at runtime—it only exists to make the compiler check your work.

### Purposes

- **To attach one or more interface contracts** to a class for compile-time verification.
- **To document intent** explicitly, making the class’s contractual obligations visible in the signature.
- **To enable safe polymorphism** by guaranteeing that the class satisfies the interface type.
- **To support multiple behavioral contracts** in a single class without multiple class inheritance.

### Syntax Rules and Structure

**General syntax:**

```
class ClassName [extends Base] implements Interface1, Interface2 {
  // members
}
```

**Component breakdown:**

- `implements` — keyword introducing the contract list.
- `Interface1, Interface2` — comma-separated type references.
- The class body must satisfy all required members of every listed interface.

**Syntax rules.**

- `implements` may appear only after the class name and optional `extends` clause.
- The list may contain any number of interfaces.
- Each listed name must resolve to a type that the class’s instance shape can satisfy.
- `implements` does **not** create a prototype link; it is a check, not an inheritance mechanism.

**Constraints and limitations.**

- The `implements` clause is **erased** and has no runtime effect.
- The class must provide concrete implementations; interfaces supply no method bodies.
- A class cannot implement an interface that requires a member the class cannot legally declare (e.g., a `private` interface member unless the class has a compatible `private` member).
- `implements` cannot be used on interfaces themselves; interfaces use `extends` to compose.

### Annotated Complete Step by Step Code Examples

#### Example 1: `implements` with `extends`

```typescript
// example_implements_extends.ts
class Animal {
  move() {
    console.log("Moving");
  }
}

interface Named {
  name: string;
}

class Dog extends Animal implements Named {
  name = "Rex";
}

const dog = new Dog();
dog.move();          // Inherited from Animal
console.log(dog.name); // Expected: Rex
```

**Expected Output:**
```
Moving
Rex
```

**Why this output occurs.** `Dog` extends `Animal` (inheriting `move`) and implements `Named` (providing `name`). The compiler checks that `Dog`’s public shape includes `name: string`. The `implements` clause emits no code; the runtime behavior comes entirely from the class and its base class.

---

#### Example 2: `implements` with an object type literal

```typescript
// example_implements_object_type.ts
interface Point {
  x: number;
  y: number;
}

class Coordinate implements Point {
  x = 0;
  y = 0;

  translate(dx: number, dy: number): void {
    this.x += dx;
    this.y += dy;
  }
}

const coord = new Coordinate();
coord.translate(3, 4);
console.log(coord.x, coord.y); // Expected: 3 4
```

**Expected Output:**
```
3 4
```

**Why this output occurs.** `Coordinate` implements the `Point` object type. It provides `x` and `y` as public fields, satisfying the contract. The additional `translate` method is allowed because `implements` requires **at least** the interface members, not exactly those members.

---

### Real-World Cases

- **Angular lifecycle hooks**: `class AppComponent implements OnInit, OnDestroy { ... }`.
- **Strategy pattern**: `class CreditCardStrategy implements PaymentStrategy { ... }`.
- **Test doubles**: `class FakeLogger implements ILogger { ... }` in unit tests.

---

## 4. Multiple Interfaces

### Definitions

**Core Definition.** **Multiple interface implementation** is the capability of a single class to implement **two or more interfaces simultaneously**, committing to satisfy the combined contract of all listed interfaces. It is TypeScript’s primary mechanism for composing behavioral contracts without multiple class inheritance.

**Technical Definition.** The `implements` clause accepts a comma-separated list of interface references: `class C implements A, B, C_ { ... }`. The compiler independently checks the class’s public instance type against each listed interface. The class must provide every required member of every interface. Members shared across interfaces must have compatible types; if two interfaces declare the same member with incompatible types, the class cannot satisfy both, and the compiler reports an error. Interfaces themselves may also **extend multiple interfaces**, composing their contracts into a single named interface.

**Beginner-Friendly Explanation.** Multiple interfaces let a class **wear several hats at once**. A `Car` can be an `Alarm` (it can alert) and a `Light` (it can turn lights on and off). Instead of forcing everything into one giant interface, you define small, focused contracts and let classes implement as many as they need. This keeps contracts clean and flexible.

### Purposes

- **To compose focused contracts** so a class can satisfy several independent behavioral requirements.
- **To avoid bloated “god interfaces”** by keeping each interface small and cohesive (Interface Segregation Principle).
- **To enable flexible polymorphism** where different combinations of capabilities are needed.
- **To work around TypeScript’s single class inheritance** by composing behavior through contracts.

### Syntax Rules and Structure

**General syntax (class implementing multiple interfaces):**

```
class ClassName implements Interface1, Interface2, Interface3 {
  // must satisfy all required members of all listed interfaces
}
```

**Component breakdown:**

- `implements` — keyword.
- `Interface1, Interface2, Interface3` — comma-separated interface references.
- Class body — must provide all required members of every listed interface.

**Interface composition syntax (interface extending multiple interfaces):**

```
interface Combined extends Interface1, Interface2 {
  // additional members
}
```

**Syntax rules.**

- Any number of interfaces may be listed after `implements`, separated by commas.
- The class must satisfy **every** required member of **every** listed interface.
- If the same member name appears in multiple interfaces, its types must be compatible; otherwise, the class cannot satisfy both.
- Interfaces may also extend multiple interfaces, creating a combined contract.

**Constraints and limitations.**

- Conflicting member types across interfaces make implementation impossible without type assertions or interface redesign.
- Implementing multiple interfaces does **not** provide multiple inheritance of implementation; the class must write all method bodies.
- Runtime behavior is unaffected; all interface information is erased.

### Annotated Complete Step by Step Code Examples

#### Example 1: Class implementing two interfaces

```typescript
// example_multiple_interfaces.ts
interface Alarm {
  alert(): void;
}

interface Light {
  lightOn(): void;
  lightOff(): void;
}

class Car implements Alarm, Light {
  alert() {
    console.log("Car alert");
  }
  lightOn() {
    console.log("Car light on");
  }
  lightOff() {
    console.log("Car light off");
  }
}

const car = new Car();
car.alert();    // Expected: Car alert
car.lightOn();  // Expected: Car light on
car.lightOff(); // Expected: Car light off
```

**Expected Output:**
```
Car alert
Car light on
Car light off
```

**Why this output occurs.** `Car` implements both `Alarm` and `Light`. The compiler checks that all three required methods (`alert`, `lightOn`, `lightOff`) are present. The class satisfies both contracts, demonstrating how a single class can fulfill multiple independent behavioral requirements.

---

#### Example 2: Interface extending multiple interfaces

```typescript
// example_interface_multi_extends.ts
interface Alarm {
  alert(): void;
}

interface Light {
  lightOn(): void;
  lightOff(): void;
}

// Combine contracts into one named interface.
interface LightableAlarm extends Alarm, Light {
  // inherits alert, lightOn, lightOff
}

class AdvancedCar implements LightableAlarm {
  alert() {
    console.log("AdvancedCar alert");
  }
  lightOn() {
    console.log("AdvancedCar light on");
  }
  lightOff() {
    console.log("AdvancedCar light off");
  }
}

const car = new AdvancedCar();
car.alert(); // Expected: AdvancedCar alert
```

**Expected Output:**
```
AdvancedCar alert
```

**Why this output occurs.** `LightableAlarm` extends both `Alarm` and `Light`, composing their contracts. `AdvancedCar` implements the combined interface, so it must provide all three methods. This pattern keeps the individual interfaces focused while allowing composition into a single contract.

---

#### Example 3: Conflicting member types prevent implementation

```typescript
// example_multiple_conflict.ts
interface A {
  value: string;
}

interface B {
  value: number;
}

// ❌ Class cannot satisfy both A and B.
// class C implements A, B {
//   value: string | number; // Error: not assignable to both.
// }

console.log("Conflicting interfaces cannot be implemented together");
```

**Expected Output:**
```
Conflicting interfaces cannot be implemented together
```

**Why this output occurs.** `A` requires `value: string` and `B` requires `value: number`. No single property type can satisfy both (a union `string | number` is not assignable to either individually). The compiler rejects the implementation. This illustrates the need for cohesive, non-conflicting interface design.

---

### Real-World Cases

- **UI components**: `class Button implements Clickable, Focusable, Disableable { ... }`.
- **Domain entities**: `class Order implements Identifiable, Timestamped, Auditable { ... }`.
- **Node.js streams**: `class DuplexStream implements Readable, Writable { ... }`.
- **Plugin systems**: a plugin implements `Configurable`, `Executable`, and `Disposable`.

---

## 5. Interface Implementation Best Practices

### Definitions

**Core Definition.** **Interface implementation best practices** are guidelines for designing and using interfaces and their class implementations in ways that maximize **clarity, type safety, flexibility, and maintainability**, while avoiding bloated contracts, fragile implementations, and misuse of `implements` where composition or abstract classes would be more appropriate.

**Technical Definition.** Best practices encompass **interface segregation** (many small, focused interfaces rather than one large interface), **composition over inheritance**, **preferring `interface` over `type` for public object contracts** (because interfaces produce clearer errors and support declaration merging), **using `readonly` for immutable fields**, **marking optional members with `?`**, **avoiding conflicting member types across implemented interfaces**, **using `implements` as an explicit intent declaration even when structural typing would suffice**, and **keeping interfaces free of implementation details**. TypeScript’s official guidance recommends interfaces for public API contracts and object shapes, reserving `type` for unions, intersections, and mapped/conditional types.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep interfaces clean and useful. The biggest rules are: keep interfaces **small and focused** (one job per interface); prefer **many small interfaces over one giant one**; use `readonly` for things that should not change; use `?` for things that are optional; and do not cram unrelated requirements into a single contract. Also, use `implements` to make your intent explicit—even though TypeScript can check shapes without it, the keyword documents what the class is promising to do.

### Purposes

- **To keep contracts cohesive and understandable** through interface segregation.
- **To prevent bloated interfaces** that force classes to implement irrelevant members.
- **To improve error messages and tooling** by preferring `interface` for public object contracts.
- **To make intent explicit** with the `implements` keyword, even when structural typing would allow implicit conformance.
- **To avoid conflicts** by designing interfaces whose shared members have compatible types.
- **To document immutability and optionality** with `readonly` and `?`.

### Syntax Rules and Structure

Best practices are conventions that interact with interface syntax. The core rules are:

1. **Interface segregation.** Create many small, focused interfaces rather than one large interface. A class should not be forced to implement methods it does not need.
2. **Prefer `interface` over `type` for object contracts.** Interfaces produce clearer compiler errors, support declaration merging, and are the recommended choice for public API object shapes. Use `type` for unions, intersections, primitives, and mapped/conditional types.
3. **Use `readonly` for immutable data.** Mark fields that should not change after creation as `readonly`.
4. **Use `?` for optional members.** Do not make members required if they are genuinely optional; use `?` to reflect reality.
5. **Make `implements` explicit.** Even though structural typing allows implicit conformance, writing `implements` documents the class’s intent and makes the contract visible.
6. **Avoid conflicting member types.** When implementing multiple interfaces, ensure shared member names have compatible types.
7. **Keep interfaces free of implementation details.** Interfaces should describe **what** is required, not **how** it is done.

**Component breakdown (interface design decision tree):**

```
Is the type a pure object shape used as a public contract?
  ├── Yes → Prefer interface.
  │        ├── Does it have many unrelated requirements?
  │        │   ├── Yes → Split into focused interfaces (segregation).
  │        │   └── No  → Keep as a single interface.
  │        └── Are there optional or readonly members?
  │            ├── Yes → Mark them with `?` or `readonly`.
  │            └── No  → Proceed.
  └── No → Is it a union, intersection, primitive, tuple, or mapped type?
           ├── Yes → Use type.
           └── No  → Re-evaluate; interface is usually the better choice for objects.
```

**Constraints and limitations.**

- Interface segregation can lead to many small interfaces; organize them clearly.
- Declaration merging can be surprising; avoid accidental merges by using `type` where merging is not desired.
- `readonly` is shallow; it does not deep-freeze nested objects.
- Optional members shift `undefined` handling to the consumer; ensure this is intentional.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface segregation — many focused interfaces

```typescript
// example_best_segregation.ts
// ❌ Bloated interface: forces unrelated requirements.
// interface Worker {
//   work(): void;
//   eat(): void;
//   sleep(): void;
// }

// ✅ Segregated interfaces: each focused on one capability.
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}

// A robot only needs Workable.
class Robot implements Workable {
  work(): void {
    console.log("Robot working");
  }
}

// A human implements all three.
class Human implements Workable, Eatable, Sleepable {
  work() { console.log("Human working"); }
  eat() { console.log("Human eating"); }
  sleep() { console.log("Human sleeping"); }
}

new Robot().work(); // Expected: Robot working
new Human().eat();  // Expected: Human eating
```

**Expected Output:**
```
Robot working
Human eating
```

**Why this output occurs.** The segregated interfaces allow `Robot` to implement only `Workable` without being forced to implement `eat` and `sleep`. `Human` implements all three. This satisfies the **Interface Segregation Principle**: no class is forced to depend on methods it does not use.

---

#### Example 2: Explicit `implements` for intent, `readonly` and `?` for precision

```typescript
// example_best_explicit_readonly.ts
interface User {
  readonly id: string;
  name: string;
  email?: string;
}

class Admin implements User {
  readonly id: string;
  name: string;
  email?: string;
  permissions: string[] = [];

  constructor(id: string, name: string) {
    this.id = id;
    this.name = name;
  }
}

const admin = new Admin("a1", "Ada");
console.log(admin.name); // Expected: Ada

// admin.id = "a2"; // Error: Cannot assign to 'id' because it is a read-only property.
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `Admin` explicitly declares `implements User`, documenting its intent. `id` is `readonly`, preventing reassignment. `email` is optional, so `Admin` is not required to provide it. The compiler verifies the contract; the interface is erased at runtime.

---

### Real-World Cases

- **SOLID design**: Interface Segregation Principle applied through small, focused interfaces.
- **Dependency injection**: services depend on narrow interfaces (`ILogger`, `IEmailSender`) rather than broad contracts.
- **Public library APIs**: interfaces provide stable, mergeable contracts with clear error messages.
- **Testing**: mocks implement only the small interfaces they need, reducing test setup.

---

## References

- TypeScript Handbook — Object Types (Current) - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook — Interfaces (Deprecated but Informative) - https://www.typescriptlang.org/docs/handbook/interfaces.html
- TypeScript Handbook — Classes (Implements Clauses) - https://www.typescriptlang.org/docs/handbook/2/classes.html
- TypeScript Handbook — Type Compatibility (Structural Subtyping) - https://www.typescriptlang.org/docs/handbook/type-compatibility.html
- Contentful — TypeScript interfaces: How-to guide, best practices, and code examples - https://www.contentful.com/fr/blog/typescript-interfaces/
- GitHub (Thomashighbaugh/opencode) — TypeScript Interface vs Type Best Practices - https://github.com/Thomashighbaugh/opencode/blob/main/skills/typescript-interface-vs-type/SKILL.md
- Convex — Interfaces in TypeScript (Multiple Interface Implementation) - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-interface
- GeeksforGeeks — How to use Interface with Class in TypeScript (Multiple Interfaces) - https://origin.geeksforgeeks.org/how-to-use-interface-with-class-in-typescript/
- Microsoft/TypeScript Issue #5749 — Implementation of Interface Requires Members Redeclaration - https://github.com/microsoft/TypeScript/issues/5749
- LogRocket Blog — Understanding and Using Interfaces in TypeScript (Granular Interfaces) - https://blog.logrocket.com/understanding-using-interfaces-in-typescript/