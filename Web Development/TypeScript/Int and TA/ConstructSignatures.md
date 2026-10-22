# Construct Signatures in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **construct signature** is a type-level construct within an object type that declares the object to be **constructable with the `new` operator**, specifying the constructor's parameter types and the type of instance produced. It enables TypeScript to describe **constructor functions and class values** as first-class types, allowing them to be passed as arguments, stored in variables, and constrained in generic functions.

**Technical Definition.** In TypeScript's grammar, a construct signature of the form `new (p₁: T₁, p₂: T₂, …): R` within an object type makes the containing type a **constructor type**—a type whose values can be invoked with `new`. The return type `R` is the **instance type** produced by the constructor. Construct signatures may be **generic** (`new <T>(arg: T): Box<T>`), **abstract** (`abstract new () => T`, since TypeScript 4.2), and may **overload** `new` operations by appearing multiple times in a single type. Construct signatures are **erased** during compilation and have no runtime representation.

**Beginner-Friendly Explanation.** A construct signature is **a way to describe the “shape” of a constructor**. Just as a function type describes what a function takes and returns, a construct signature describes what a **class or constructor function** takes and what kind of object it creates. It lets you say: “This value is something I can call with `new`, it takes these arguments, and it produces this type of object.” This is essential for factory functions, dependency injection, and generic utilities that accept class constructors.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`new` keyword prefix** | Construct signatures are written by placing `new` in front of a call signature: `new (args): InstanceType`. |
| **Instance type return** | The return type is the **constructed instance type**, not the constructor itself. |
| **Generic support** | Construct signatures may declare type parameters: `new <T>(arg: T): Box<T>`. |
| **Abstract construct signatures** | Since TypeScript 4.2, `abstract new () => T` describes constructors of abstract classes that cannot be directly instantiated. |
| **Overload support** | Multiple construct signatures in one type overload the `new` operation. |
| **Call + construct combination** | A type may combine call and construct signatures to describe values callable both with and without `new`. |
| **Erasure** | Construct signatures exist only at compile time and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: interfaces, type aliases, and object types.
- **Function type expressions** and **call signatures**.
- **Classes**: constructors, `new`, and instance types.
- **Generics fundamentals**: type parameters and constraints (`<T extends ...>`).
- **`typeof` operator** for referencing class constructor types.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Factory patterns** and **dependency injection** — accepting constructors as parameters.
- **Mixins** — constraining base constructors in generic mixin functions.
- **Generic utilities** — `InstanceType<T>` and `ConstructorParameters<T>` rely on construct signatures.
- **Abstract classes** — abstract construct signatures model abstract constructors.
- **Hybrid types** — combining call and construct signatures for values usable both ways.

---

## Core Concepts / Features

---

## 1. Construct Signatures

### Definitions

**Core Definition.** A **construct signature** is an unnamed member of an object type that declares the object to be constructable with `new`, specifying the constructor's parameters and the produced instance type.

**Technical Definition.** A construct signature of the form `new (param: Type): InstanceType` inside an interface or type literal makes the containing type a **constructor type**. Values of that type can be invoked with `new`, and the result is typed as `InstanceType`. Construct signatures may be generic, abstract, and overloaded. They are **not implementable by classes** in the `implements` clause; they describe **existing constructor values** such as classes and constructor functions.

**Beginner-Friendly Explanation.** A construct signature is **a description of a class or constructor function**. It says: “This thing can be used with `new`, it expects these arguments, and it creates this kind of object.” You use it to type variables, parameters, and properties that hold constructors—like passing a class to a factory function.

### Purposes

- **To type constructor functions and class values** passed as arguments or stored in variables.
- **To enable factory functions** that accept any constructor producing a compatible instance.
- **To constrain generic functions** to constructors (e.g., mixins, dependency injection).
- **To model abstract constructors** that cannot be directly instantiated.
- **To overload the `new` operation** with multiple constructor forms.

### Syntax Rules and Structure

**General syntax:**

```
type ConstructorType = new (param1: Type1, param2: Type2) => InstanceType;
```

**Component breakdown:**

- `new` — the keyword indicating a construct signature.
- `(param1: Type1, param2: Type2)` — the constructor's parameter list.
- `=>` — the return type separator (note: construct signatures in **object type literals** use `:` instead, e.g., `{ new (s: string): Date }`).
- `InstanceType` — the type of the object produced by `new`.

**Inside an interface (using `:`):**

```
interface PointConstructor {
  new (x: number, y: number): Point;
}
```

**Syntax rules.**

- Construct signatures may be **generic**: `new <T>(arg: T) => Box<T>`.
- Construct signatures may be **abstract**: `abstract new () => T` (TypeScript 4.2+).
- Multiple construct signatures in one type act as overloads.
- A type may combine call and construct signatures: `{ new (s: string): Date; (n?: number): number }`.
- The return type is the **instance type**, not the constructor type.

**Constraints and limitations.**

- **Classes cannot `implements` a construct signature**; construct signatures describe existing constructor values, not class contracts.
- Abstract construct signatures cannot be assigned to non-abstract construct signatures (though the reverse is allowed).
- Construct signatures are erased and provide no runtime reflection.
- Confusing construct signatures with call signatures is a common source of errors; use `new` for constructable values and `()` for callable values.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic construct signature for a factory function

```typescript
// Step 1: Define an interface with a construct signature.
interface PointConstructor {
  new (x: number, y: number): { x: number; y: number };
}

// Step 2: Define a class that satisfies the construct signature.
class Point {
  constructor(public x: number, public y: number) {}
}

// Step 3: Write a factory that accepts any conforming constructor.
function createPoint(Ctor: PointConstructor, x: number, y: number) {
  return new Ctor(x, y);
}

// Step 4: Use the factory with the class.
const p = createPoint(Point, 1, 2);
console.log(p.x, p.y); // Expected: 1 2
```

**Setup.** Compile with `npx tsc example_construct_basic.ts --strict`.

**Expected Output:**
```
1 2
```

**Why this output occurs.** The `PointConstructor` interface declares a construct signature that takes two numbers and produces an object with `x` and `y`. The `Point` class satisfies this signature because `new Point(x, y)` produces a `Point` with those properties. The factory function accepts any conforming constructor, enabling dependency injection of constructor values.

---

#### Example 2: Generic construct signature

```typescript
// A generic constructor type: builds a Box<T> from a value.
type BoxConstructor = new <T>(value: T) => { value: T };

class Box<T> {
  constructor(public value: T) {}
}

const createBox: BoxConstructor = Box;

const stringBox = createBox("hello");
const numberBox = createBox(42);

console.log(stringBox.value); // Expected: hello
console.log(numberBox.value); // Expected: 42
```

**Expected Output:**
```
hello
42
```

**Why this output occurs.** The construct signature `new <T>(value: T) => { value: T }` declares a generic constructor that infers `T` from the argument. The `Box` class is generic and matches this signature. Each call infers `T` independently, producing correctly typed instances.

---

#### Example 3: Abstract construct signature

```typescript
// Step 1: Declare an abstract class.
abstract class Animal {
  abstract speak(): string;
}

// Step 2: Use abstract new to describe its constructor type.
type AnimalConstructor = abstract new () => Animal;

// Step 3: A function that accepts abstract constructors.
function createAnimal(Ctor: AnimalConstructor): Animal {
  return new Ctor(); // ❌ Error: Cannot create an instance of an abstract class.
}

// Step 4: Subclass provides a concrete constructor.
class Dog extends Animal {
  speak() {
    return "Woof";
  }
}

// The non-abstract Dog constructor is assignable to the abstract type.
const dogCtor: AnimalConstructor = Dog;

console.log("Abstract construct signature declared");
```

**Expected Output:**
```
Abstract construct signature declared
```

**Why this output occurs.** `AnimalConstructor` uses `abstract new () => Animal`, which means “a constructor for `Animal` that may be abstract.” You cannot call `new Ctor()` on an abstract construct signature—the compiler rejects it. The non-abstract `Dog` constructor **is** assignable to the abstract construct signature because non-abstract constructors are assignable to abstract ones (the reverse is not true). This pattern is used in mixin factories that accept abstract base classes.

---

#### Example 4: Combined call and construct signatures

```typescript
// A value that can be called with or without new.
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}

function fn(input?: string | number): Date | number {
  if (typeof input === "string") return new Date(input);
  return input ?? 0;
}

const hybrid: CallOrConstruct = fn as CallOrConstruct;

const date = new hybrid("2025-01-01");
const num = hybrid(42);

console.log(date instanceof Date); // Expected: true
console.log(num);                  // Expected: 42
```

**Expected Output:**
```
true
42
```

**Why this output occurs.** The `CallOrConstruct` interface combines a construct signature (`new (s: string): Date`) with a call signature (`(n?: number): number`). The function is asserted to the hybrid type and used both with `new` and as a normal call. The compiler checks both usage forms. This pattern models JavaScript APIs that behave differently when called with or without `new`.

---

### Real-World Cases

- **Factory functions**: `createPoint(Point, 1, 2)` where `Point` is a constructor value.
- **Dependency injection**: `constructor(service: new () => Service) { this.service = new service(); }`.
- **Mixins**: constraining `TBase extends abstract new (...args: any[]) => {}` to accept abstract base classes.
- **Generic utilities**: `InstanceType<T>` and `ConstructorParameters<T>` are built on construct signatures.

---

## 2. `new () => T` Syntax

### Definitions

**Core Definition.** The syntax **`new () => T`** is a **constructor type** expressed as a type alias, where `T` is the instance type produced by calling `new` on the constructor with no arguments. It is the most concise form of a construct signature.

**Technical Definition.** The type `new () => T` is a constructor type with zero parameters and return type `T`. It is the canonical shape of a **no-argument constructor**. With parameters, it becomes `new (...args: any[]) => T` or a specific parameter list. Since TypeScript 4.2, `abstract new () => T` describes abstract constructors that cannot be instantiated directly but can be passed to mixin functions. The return type `T` is the **instance type**, not the constructor type.

**Beginner-Friendly Explanation.** `new () => T` means **“a constructor that takes no arguments and creates a `T`.”** It is a shorthand for describing a class or constructor function. If the constructor takes arguments, you write them inside the parentheses: `new (name: string) => User`. If the class is abstract, you write `abstract new () => T`.

### Purposes

- **To describe no-argument constructors** concisely.
- **To constrain generic functions** to constructors producing a specific type.
- **To type factory functions and dependency injection** parameters.
- **To model abstract constructors** with `abstract new () => T`.

### Syntax Rules and Structure

**General syntax:**

```
type Constructor = new () => T;
```

**Component breakdown:**

- `new` — construct signature keyword.
- `()` — empty parameter list (no arguments).
- `=>` — return type separator.
- `T` — the instance type produced.

**With parameters:**

```
type Constructor = new (name: string, age: number) => T;
```

**Abstract form:**

```
type AbstractConstructor = abstract new () => T;
```

**Syntax rules.**

- `new () => T` is a **type expression**, not a runtime value.
- The return type is always the **instance type**, never the constructor type.
- Abstract construct signatures must use the arrow form: `abstract new () => T`; they cannot be written as `new abstract ()`.
- `new () => T` may be used inline in generic constraints: `<T extends new () => any>`.

**Constraints and limitations.**

- A non-abstract class constructor is assignable to an abstract construct signature, but not vice versa.
- Values with abstract construct signature types cannot be instantiated with `new`.
- The `new () => T` syntax cannot describe callable values; use call signatures for that.

### Annotated Complete Step by Step Code Examples

#### Example 1: `new () => T` in a factory

```typescript
// example_new_arrow.ts
// Step 1: Define a constructor type.
type Constructor = new () => { id: string };

// Step 2: Create a class with a no-arg constructor.
class Entity {
  id = "default";
}

// Step 3: Factory accepts the constructor type.
function create(Ctor: Constructor) {
  return new Ctor();
}

const entity = create(Entity);
console.log(entity.id); // Expected: default
```

**Expected Output:**
```
default
```

**Why this output occurs.** `Constructor` is `new () => { id: string }`. The `Entity` class has a no-argument constructor and produces an object with `id`. The factory function accepts any such constructor and instantiates it. The return type is correctly inferred as `{ id: string }`.

---

#### Example 2: Generic constraint with `new () => T`

```typescript
// example_new_arrow_generic.ts
// Constrain T to be a constructor producing a value with `name`.
function createNamed<T extends new () => { name: string }>(Ctor: T) {
  const instance = new Ctor();
  instance.name = "Ada"; // Allowed: instance has a name property.
  return instance;
}

class Person {
  name = "";
}

const person = createNamed(Person);
console.log(person.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The generic constraint `T extends new () => { name: string }` ensures that `Ctor` is a no-argument constructor producing an object with a `name` property. Inside the function, `instance` is typed as having `name`, so assignment is allowed. The return type is the specific `Person` type, not just `{ name: string }`.

---

#### Example 3: Abstract `new () => T` in a mixin

```typescript
// example_new_arrow_abstract.ts
// Abstract construct signature allows abstract base classes.
type AbstractConstructor = abstract new (...args: any[]) => {};

function Timestamped<TBase extends AbstractConstructor>(Base: TBase) {
  return class extends Base {
    timestamp = new Date();
  };
}

abstract class Entity {
  abstract id: string;
}

// ✅ Works: Entity is abstract and accepted by abstract new.
class TimestampedEntity extends Timestamped(Entity) {
  id = "e1";
}

const te = new TimestampedEntity();
console.log(te.timestamp instanceof Date); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The mixin constraint uses `abstract new (...args: any[]) => {}`, which accepts abstract constructors. Without `abstract`, passing the abstract `Entity` class would be a compile-time error because abstract constructors are not assignable to non-abstract construct signatures. This pattern is essential for composing mixins with abstract base classes.

---

### Real-World Cases

- **Generic factories**: `function create<T>(Ctor: new () => T): T`.
- **Dependency injection**: `constructor(private readonly service: new () => Service) {}`.
- **Mixins with abstract bases**: `TBase extends abstract new (...args: any[]) => {}`.
- **Type utilities**: `InstanceType<typeof C>` relies on the construct signature of `C`.

---

## 3. Construct Signature Best Practices

### Definitions

**Core Definition.** **Construct signature best practices** are guidelines for using construct signatures and constructor types in TypeScript in ways that maximize **type safety, clarity, and maintainability**, while avoiding common pitfalls such as confusing call and construct signatures, misusing abstract constructors, and over-constraining generic constructors.

**Technical Definition.** Best practices center on **using `new` for constructable values and `()` for callable values**, **distinguishing abstract from non-abstract constructors**, **constraining constructors precisely** (`new () => T` vs. `new (...args: any[]) => T`), **preferring `InstanceType<T>` for extracting instance types**, **using `ConstructorParameters<T>` for parameter typing**, and **avoiding construct signatures in `implements` clauses** (classes cannot implement them). When accepting abstract constructors, use `abstract new () => T` rather than `new () => T`.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep construct signatures clear and safe. The big rules are: use `new` when you mean “can be constructed,” use `()` when you mean “can be called”; use `abstract new` when you need to accept abstract classes; and constrain constructors as narrowly as possible so you get the strongest type safety.

### Purposes

- **To distinguish constructable from callable values** clearly.
- **To correctly accept abstract and non-abstract constructors** without type errors.
- **To maximize type safety** by constraining constructors to the exact instance type and parameters needed.
- **To leverage built-in utilities** (`InstanceType<T>`, `ConstructorParameters<T>`) instead of manual extraction.
- **To avoid the common mistake** of trying to `implements` a construct signature.

### Syntax Rules and Structure

Best practices are conventions that interact with construct signature syntax. The core rules are:

1. **Use `new` for constructable values, `()` for callable values.** Do not mix them up; `new () => T` describes a constructor, while `() => T` describes a function.
2. **Use `abstract new () => T` to accept abstract constructors.** Non-abstract constructors are assignable to abstract construct signatures, but not vice versa.
3. **Constrain constructors precisely.** Use `new () => T` for no-argument constructors and `new (param: Type) => T` for specific parameters. Avoid `new (...args: any[]) => T` unless the parameters are genuinely unknown.
4. **Prefer `InstanceType<T>` for extracting instance types.** `InstanceType<typeof C>` is the canonical way to get the instance type of a class `C`.
5. **Use `ConstructorParameters<T>` for constructor parameter types.** This utility extracts the parameter tuple from a constructor type.
6. **Do not use construct signatures in `implements` clauses.** Classes cannot implement construct signatures; they describe existing constructor values, not class contracts.
7. **Combine call and construct signatures deliberately.** When a value is both callable and constructable (like `Date`), combine both signatures in one type.

**Component breakdown (construct signature decision tree):**

```
Does the value need to be instantiated with `new`?
  ├── Yes → Use a construct signature (`new () => T` or `new (args) => T`).
  │        ├── Is the constructor abstract? → Use `abstract new () => T`.
  │        └── Are the parameters known? → Use specific parameter types.
  └── No → Does the value need to be called as a function?
           ├── Yes → Use a call signature (`() => T`).
           └── No → Re-evaluate; the value may be a plain object.
```

**Constraints and limitations.**

- Abstract construct signatures cannot be instantiated with `new`; they can only be passed to code that accepts them.
- `new (...args: any[]) => T` loses parameter type safety; prefer specific parameters when possible.
- Construct signatures cannot be implemented by classes; use them only to type existing constructor values.
- Confusing `new () => T` with `() => T` is a common source of runtime errors.

### Annotated Complete Step by Step Code Examples

#### Example 1: Precise constructor constraint

```typescript
// example_best_precise.ts
// ❌ Overly broad: loses parameter type safety.
// type BroadConstructor = new (...args: any[]) => User;

// ✅ Precise: enforces the exact constructor parameters.
type UserConstructor = new (name: string, age: number) => User;

class User {
  constructor(public name: string, public age: number) {}
}

function createUser(Ctor: UserConstructor, name: string, age: number) {
  return new Ctor(name, age);
}

const user = createUser(User, "Ada", 36);
console.log(user.name, user.age); // Expected: Ada 36
```

**Expected Output:**
```
Ada 36
```

**Why this output occurs.** The construct signature specifies that the constructor must take exactly `(string, number)`. The `User` class satisfies this. The factory function gets full type checking on both the constructor and its arguments, unlike the broad `any[]` version.

---

#### Example 2: Using `InstanceType` and `ConstructorParameters`

```typescript
// example_best_utility.ts
class Point {
  constructor(public x: number, public y: number) {}
}

// Extract the instance type and constructor parameters.
type PointInstance = InstanceType<typeof Point>;
type PointParams = ConstructorParameters<typeof Point>;

const p: PointInstance = new Point(1, 2);
const args: PointParams = [3, 4];

console.log(p.x);    // Expected: 1
console.log(args[0]); // Expected: 3
```

**Expected Output:**
```
1
3
```

**Why this output occurs.** `InstanceType<typeof Point>` extracts the instance type `Point` from the constructor type `typeof Point`. `ConstructorParameters<typeof Point>` extracts the parameter tuple `[number, number]`. These utilities avoid manual construct signature extraction and are the canonical way to work with constructor types.

---

#### Example 3: Abstract vs. non-abstract in mixins

```typescript
// example_best_abstract_mixin.ts
type ConcreteConstructor = new (...args: any[]) => {};
type AbstractConstructor = abstract new (...args: any[]) => {};

// ✅ Mixin accepts abstract constructors.
function Serializable<TBase extends AbstractConstructor>(Base: TBase) {
  return class extends Base {
    serialize() {
      return JSON.stringify(this);
    }
  };
}

abstract class Shape {
  abstract area(): number;
}

class Square extends Serializable(Shape) {
  side = 4;
  area() {
    return this.side ** 2;
  }
}

const sq = new Square();
console.log(sq.area());         // Expected: 16
console.log(typeof sq.serialize()); // Expected: string
```

**Expected Output:**
```
16
string
```

**Why this output occurs.** The mixin constraint uses `abstract new (...args: any[]) => {}`, which accepts both abstract and non-abstract constructors. Passing the abstract `Shape` class would fail with a non-abstract constraint. The resulting class `Square` inherits the `serialize` method from the mixin and implements the abstract `area` method.

---

### Real-World Cases

- **Dependency injection containers**: accepting `new () => Service` for service registration.
- **ORM factories**: `new (data: Partial<T>) => T` for model instantiation.
- **Mixin libraries**: `TBase extends abstract new (...args: any[]) => {}` to support abstract base classes.
- **Type utilities**: `InstanceType<T>` and `ConstructorParameters<T>` built on construct signatures.

---

## References

- TypeScript Handbook — More on Functions: Construct Signatures - https://www.typescriptlang.org/docs/handbook/2/functions.html#construct-signatures
- TypeScript Handbook — More on Functions: Combining Call and Construct Signatures - https://www.typescriptlang.org/docs/handbook/2/functions.html#construct-signatures
- Microsoft/TypeScript Pull Request #36392 — Add Support for Abstract Constructor Types - https://github.com/microsoft/TypeScript/pull/36392
- TypeScript 4.2 Release Notes — Abstract Construct Signatures - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-2.html
- GeeksforGeeks — TypeScript Construct Signatures - https://www.geeksforgeeks.org/typescript/typescript-construct-signatures/
- Stack Overflow — How does interfaces with construct signatures work? - https://stackoverflow.com/questions/13407036/how-does-interfaces-with-construct-signatures-work
- Stack Overflow — What is new() in TypeScript? - https://stackoverflow.com/questions/39622778/what-is-new-in-typescript
- Stack Overflow — How to implement TypeScript construct signature containing both constructor and methods? - https://stackoverflow.com/questions/68350115/how-to-implement-typescript-construct-signature-containing-both-constructor-and
- TypeScript ESLint — `no-misused-new` Rule - https://typescript-eslint.io/rules/no-misused-new/
- TypeScript Handbook — Utility Types: `InstanceType<T>` - https://www.typescriptlang.org/docs/handbook/utility-types.html#instancetypetype
- TypeScript Handbook — Utility Types: `ConstructorParameters<T>` - https://www.typescriptlang.org/docs/handbook/utility-types.html#constructorparameterstype
- TypeScript Language Specification — Construct Signatures - https://raw.githubusercontent.com/microsoft/TypeScript/refs/tags/v2.3.3/doc/spec.md
- Convex — TypeScript Construct Signatures Guide - https://www.convex.dev/typescript/advanced/type-operators-manipulation/typescript-construct-signature