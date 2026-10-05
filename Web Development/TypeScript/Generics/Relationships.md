# TypeScript Generic Relationships: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Generic relationships in TypeScript describe how multiple type parameters interact with each other, how generic types relate to their subtypes, and how generic type parameters can be used to model complex interdependencies between types. These relationships form the foundation for advanced generic programming, enabling type-safe factories, callbacks, and variance-controlled abstractions.

**Technical Definition**
TypeScript's generic system supports several kinds of relationships between type parameters. Multiple type parameters can be linked through constraints (e.g., `K extends keyof T`), establishing a dependency where one parameter's valid values depend on another parameter's type. Generic instances relate to each other through variance—a property describing how subtyping of type arguments propagates to subtyping of generic types. Variance can be inferred by the compiler from usage, or explicitly annotated using `in` and `out` modifiers (TypeScript 4.7+). Generic callbacks and higher-order parameter mapping enable contextual typing to flow through generic function signatures. Generic factories use constructor signatures (`new (...args: any[]) => T`) to enable type-safe instantiation of generic types.

**Beginner-Friendly Explanation**
Generic relationships are about how different type parameters work together. Sometimes you have two type parameters where one depends on the other—like a key that must be a valid property of an object. Sometimes you want to control how generic types relate to each other when subtyping is involved—this is called variance. TypeScript can figure out variance automatically, but you can also annotate it explicitly with `in` and `out`. Generic callbacks let TypeScript infer parameter types from context, and generic factories let you create objects of any type using their constructor. Understanding these relationships is key to writing advanced, reusable TypeScript code.

### Key Characteristics

- **Inter-dependent type parameters**: Type parameters can be constrained by each other (`K extends keyof T`).
- **Variance**: Generic types relate to their subtypes through covariance, contravariance, bivariance, or invariance.
- **Inferred variance**: TypeScript infers variance from how type parameters are used.
- **Explicit variance annotations**: `in`, `out`, and `in out` modifiers (TypeScript 4.7+) let authors declare intended variance.
- **Contextual typing**: Generic callbacks receive parameter types from their usage context.
- **Constructor signatures**: `new (...args: any[]) => T` enables generic factory patterns.
- **Compile-time only**: All generic relationships are erased at runtime.

### Prerequisites

- Solid understanding of TypeScript generics fundamentals
- Familiarity with generic constraints (`extends`)
- Understanding of structural typing and subtyping
- Familiarity with `keyof`, indexed access types (`T[K]`), and mapped types
- Basic knowledge of function types and callbacks

### Related Programming Areas

- **Type Theory**: Variance, bounded quantification, type constructors
- **Type-Level Programming**: Conditional types, mapped types, `infer`
- **Design Patterns**: Factory, strategy, observer, and dependency injection
- **Library Design**: Building reusable generic APIs
- **Object-Oriented Programming**: Generic classes and inheritance

### Core Concepts / Features

1. Multiple Type Parameters and Inter-Dependencies (`<T, K extends keyof T>`)
2. Subtyping, Compatibility, and Type Relationships Among Generic Instances
3. Variance Annotations (`in`, `out`, `in out`) for Strict Generic Type Checking
4. Generic Callbacks, Higher-Order Parameter Mapping, and Contextual Typing
5. Generic Factories and Constructor Signatures (`new (...args: any[]) => T`)


## 1. Multiple Type Parameters and Inter-Dependencies (`<T, K extends keyof T>`)

### Definitions

**Core Definition**
Multiple type parameters with inter-dependencies allow one type parameter to be constrained by another, creating a relationship where the valid values of one parameter depend on the type of another. The canonical example is `<T, K extends keyof T>`, where `K` is constrained to be a valid property name of `T`.

**Technical Definition**
When a generic declaration has multiple type parameters, later parameters can reference earlier ones in their constraints. The constraint `K extends keyof T` establishes that `K` must be assignable to the union of `T`'s property names (as computed by the `keyof` operator). This enables type-safe property access through indexed access types (`T[K]`), where the return type is the type of the property at key `K` in `T`. The relationship is structural: any type with the required properties satisfies the constraint, regardless of its declaration site. TypeScript 2.1 introduced `keyof` and indexed access types specifically to express these relationships.

**Beginner-Friendly Explanation**
Multiple type parameters with dependencies let you say "this key must be a valid property of this object." For example, `function getProperty<T, K extends keyof T>(obj: T, key: K): T[K]` takes an object `T` and a key `K` that must be one of `T`'s property names. TypeScript ensures you only use valid keys, and it knows the return type is the type of that property. This pattern is everywhere in TypeScript—`Pick`, `Omit`, and many utility types use it. It's how you write functions that work with any property of any object while staying type-safe.

### Purposes

- To establish relationships between type parameters where one depends on another.
- To enable type-safe property access using keys constrained to valid property names.
- To build generic utilities like `getProperty`, `setProperty`, `Pick`, and `Omit`.
- To prevent typos in property names at compile time.
- To preserve the specific type of a property value in generic functions.

### Syntax Rules and Structure

**General Syntax: Key-Constrained Type Parameters**

```typescript
function functionName<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

**Component Breakdown**
- `<T, K extends keyof T>`: `K` is constrained to `T`'s property names.
- `obj: T`: The object parameter.
- `key: K`: The key parameter, constrained to valid keys.
- `): T[K]`: The return type is the type of the property at `key`.

**General Syntax: Multiple Constraints**

```typescript
function functionName<T extends object, U extends keyof T>(obj: T, key: U): T[U] {
  return obj[key];
}
```

**Component Breakdown**
- Both parameters have constraints; `U` depends on `T`.

**General Syntax: Constraint with Default**

```typescript
type Pick<T, K extends keyof T = keyof T> = {
  [P in K]: T[P];
};
```

**Component Breakdown**
- `K extends keyof T = keyof T`: Defaults to all keys.

**Syntax Rules**

- Later type parameters can reference earlier ones in their constraints.
- `keyof T` produces a union of `T`'s public property names.
- `T[K]` (indexed access type) produces the type of the property at `K`.
- The constraint is checked structurally; any type with the required keys satisfies it.
- Multiple type parameters are separated by commas.
- Constraints can use any type expression, including other type parameters.

**Constraints and Limitations**

- `keyof T` on a type without properties returns `never`.
- `keyof T` on `any` returns `string | number | symbol`.
- `keyof` on a union distributes, producing the union of all keys (not the intersection).
- Index signatures affect `keyof` results (e.g., `[key: string]: T` produces `string | number`).
- Type parameters cannot be forward-referenced (later parameters cannot constrain earlier ones).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `getProperty` with Key Constraint

```typescript
// Step 1: Define an interface.
interface Person {
  name: string;
  age: number;
  email: string;
}

// Step 2: Define a function with a key-constrained type parameter.
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Step 3: Use with valid keys — return type is correctly inferred.
const person: Person = { name: "Alice", age: 30, email: "alice@example.com" };

const name = getProperty(person, "name");   // string
const age = getProperty(person, "age");     // number
const email = getProperty(person, "email"); // string

console.log(name);   // "Alice"
console.log(age);    // 30
console.log(email);  // "alice@example.com"

// Step 4: Invalid keys are compile errors.
// getProperty(person, "gender");  // ❌ Error: "gender" is not assignable to "name" | "age" | "email".

// Step 5: Type safety is preserved through the key.
const nameLength: number = getProperty(person, "name").length;  // ✅
const ageFixed: string = getProperty(person, "age").toFixed(2); // ✅
```

**Expected Output:**
```
Alice
30
alice@example.com
```

**Why This Output Occurs:** The constraint `K extends keyof T` restricts `key` to `"name" | "age" | "email"`. The return type `T[K]` is the specific type of the property at `key`. Invalid keys are caught at compile time.

#### Example 2: Multiple Inter-Dependent Parameters with Constraints

```typescript
// Step 1: Define a base entity interface.
interface Entity {
  id: number;
  name: string;
  createdAt: Date;
}

// Step 2: Function with two dependent type parameters.
function updateEntity<T extends Entity, K extends keyof T>(
  entity: T,
  key: K,
  value: T[K]
): T {
  return { ...entity, [key]: value } as T;
}

// Step 3: Use with valid key-value pairs.
const user: Entity = { id: 1, name: "Alice", createdAt: new Date() };

const updated1 = updateEntity(user, "name", "Bob");
console.log(updated1.name);  // "Bob"

const updated2 = updateEntity(user, "id", 42);
console.log(updated2.id);    // 42

// Step 4: Invalid value types are compile errors.
// updateEntity(user, "name", 42);  // ❌ Error: number not assignable to string.
// updateEntity(user, "createdAt", "2024-01-01");  // ❌ Error: string not assignable to Date.

// Step 5: Type relationships are preserved.
const updated3 = updateEntity(user, "createdAt", new Date("2025-01-01"));
console.log(updated3.createdAt instanceof Date);  // true
```

**Expected Output:**
```
Bob
42
true
```

**Why This Output Occurs:** The `K extends keyof T` constraint ensures `key` is a valid property of `T`, and `value: T[K]` ensures the value matches the property's type. TypeScript infers the relationship between `K` and `T[K]`, preventing type mismatches.

### Real-World Cases

**Case 1: Form Libraries**
Form libraries use `K extends keyof FormValues` to type field names and ensure type-safe access to form values.

**Case 2: State Management**
State management utilities use `keyof` constraints to type action creators and reducers that operate on state properties.

**Case 3: Database Query Builders**
Query builders use `keyof Entity` to type column names, preventing SQL injection and typos.

**Case 4: Configuration Access**
Configuration helpers use `keyof Config` to provide type-safe access to configuration values.

**Case 5: Data Transformation**
Data transformation utilities use `keyof` constraints to type functions that pick, omit, or transform object properties.


## 2. Subtyping, Compatibility, and Type Relationships Among Generic Instances

### Definitions

**Core Definition**
Subtyping among generic instances describes how generic types relate to each other when their type arguments are related by subtyping. For example, if `Dog` is a subtype of `Animal`, does `Box<Dog>` relate to `Box<Animal>`? The answer depends on variance—the direction in which subtyping propagates through the generic type constructor.

**Technical Definition**
TypeScript's type compatibility is based on structural subtyping: two types are compatible if their members are compatible, regardless of declaration site. For generic types, compatibility is determined by variance. Covariance means `F<T>` and `T` co-vary (if `T extends U`, then `F<T> extends F<U>`); contravariance means they contra-vary (if `T extends U`, then `F<U> extends F<T>`); invariance means neither direction works; bivariance means both directions work. TypeScript infers variance from how type parameters are used: read positions (return types) are covariant, write positions (parameter types) are contravariant, and positions used as both are invariant. Under `strictFunctionTypes`, function type parameter positions are checked contravariantly instead of bivariantly, while method parameters remain bivariant for backward compatibility with generic classes like `Array<T>`.

**Beginner-Friendly Explanation**
Subtyping among generics is about how generic types relate when their type arguments relate. If `Dog` is a subtype of `Animal`, is `Box<Dog>` a subtype of `Box<Animal>`? That depends on variance. Covariance means yes—the relationship goes in the same direction. Contravariance means no—it goes in the opposite direction. Invariance means neither works. TypeScript figures out variance automatically based on how you use the type parameter: if it's only in return types, it's covariant; if it's only in parameter types, it's contravariant; if it's in both, it's invariant. This is important for understanding why some assignments work and others don't.

### Purposes

- To understand when generic types with related type arguments are assignable.
- To predict and diagnose type compatibility errors in generic code.
- To design generic APIs with the intended variance behavior.
- To leverage TypeScript's automatic variance inference.
- To use explicit variance annotations when inference is insufficient.

### Syntax Rules and Structure

**General Syntax: Covariance (Read-Only)**

```typescript
type Producer<T> = () => T;
// T is covariant (out) — only in return position.

declare let animalProducer: Producer<Animal>;
declare let dogProducer: Producer<Dog>;

animalProducer = dogProducer;  // ✅ Dog is a subtype of Animal
// dogProducer = animalProducer;  // ❌ Error
```

**Component Breakdown**
- `Producer<T>` uses `T` only in return position.
- `Dog extends Animal` implies `Producer<Dog> extends Producer<Animal>`.

**General Syntax: Contravariance (Write-Only)**

```typescript
type Consumer<T> = (value: T) => void;
// T is contravariant (in) — only in parameter position.

declare let animalConsumer: Consumer<Animal>;
declare let dogConsumer: Consumer<Dog>;

dogConsumer = animalConsumer;  // ✅ Animal can be passed where Dog is expected
// animalConsumer = dogConsumer;  // ❌ Error
```

**Component Breakdown**
- `Consumer<T>` uses `T` only in parameter position.
- `Dog extends Animal` implies `Consumer<Animal> extends Consumer<Dog>`.

**General Syntax: Bivariance (Method Syntax)**

```typescript
interface Comparer<T> {
  compare(a: T, b: T): number;  // Method syntax — bivariant
}

declare let animalComparer: Comparer<Animal>;
declare let dogComparer: Comparer<Dog>;

animalComparer = dogComparer;  // ✅ Allowed (bivariant)
dogComparer = animalComparer;  // ✅ Allowed (bivariant)
```

**Component Breakdown**
- Method syntax parameters are bivariant, even under `strictFunctionTypes`.

**General Syntax: Invariance**

```typescript
interface Box<T> {
  get(): T;
  set(value: T): void;
}
// T is invariant — used in both return and parameter positions.
```

**Component Breakdown**
- `T` appears in both read and write positions, making it invariant.

**Syntax Rules**

- Covariance: `T` used only in return positions.
- Contravariance: `T` used only in parameter positions.
- Bivariance: `T` used in method parameter positions (legacy).
- Invariance: `T` used in both read and write positions.
- Under `strictFunctionTypes`, function type parameters are contravariant.
- Method parameters remain bivariant for backward compatibility.
- TypeScript infers variance from usage; explicit annotations override inference.

**Constraints and Limitations**

- Bivariance in method syntax is a deliberate unsoundness for practical reasons.
- Arrays are covariant in TypeScript, which is unsound for mutable arrays.
- Variance inference can fail in complex generic interfaces, producing unexpected results.
- Explicit variance annotations (TypeScript 4.7+) can catch variance errors at declaration sites.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Covariance and Contravariance

```typescript
// Step 1: Define a class hierarchy.
class Animal {
  name: string = "";
}

class Dog extends Animal {
  breed: string = "";
}

// Step 2: Covariance — read-only producer.
type Producer<T> = () => T;

declare let animalProducer: Producer<Animal>;
declare let dogProducer: Producer<Dog>;

animalProducer = dogProducer;  // ✅ Dog is an Animal
// dogProducer = animalProducer;  // ❌ Error: Animal may not be a Dog

// Step 3: Contravariance — write-only consumer (function property syntax).
type Consumer<T> = (value: T) => void;

declare let animalConsumer: Consumer<Animal>;
declare let dogConsumer: Consumer<Dog>;

dogConsumer = animalConsumer;  // ✅ Animal handler can handle Dogs
// animalConsumer = dogConsumer;  // ❌ Error: Dog handler may not handle Animals

// Step 4: Bivariance — method syntax (legacy).
interface Comparer<T> {
  compare(a: T, b: T): number;
}

declare let animalComparer: Comparer<Animal>;
declare let dogComparer: Comparer<Dog>;

animalComparer = dogComparer;  // ✅ Allowed (bivariant)
dogComparer = animalComparer;  // ✅ Allowed (bivariant)

console.log("All assignments completed.");
```

**Expected Output:**
```
All assignments completed.
```

**Why This Output Occurs:** Covariance allows `Producer<Dog>` to be assigned to `Producer<Animal>` because `Dog` is a subtype of `Animal`. Contravariance allows `Consumer<Animal>` to be assigned to `Consumer<Dog>` because an Animal handler can handle Dogs. Bivariance (method syntax) allows both directions, which is less safe but maintained for backward compatibility.

#### Example 2: Invariance and Explicit Variance Annotations

```typescript
// Step 1: Invariant generic type (both read and write).
interface Box<T> {
  get(): T;
  set(value: T): void;
}

declare let animalBox: Box<Animal>;
declare let dogBox: Box<Dog>;

// animalBox = dogBox;  // ❌ Error: Dog may not be an Animal
// dogBox = animalBox;  // ❌ Error: Animal may not be a Dog

// Step 2: Explicit variance annotations (TypeScript 4.7+).
interface Producer<out T> {
  produce(): T;
}

interface Consumer<in T> {
  consume(value: T): void;
}

interface State<in out T> {
  get(): T;
  set(value: T): void;
}

// Step 3: Annotations enforce intended variance.
declare let animalProducer: Producer<Animal>;
declare let dogProducer: Producer<Dog>;

animalProducer = dogProducer;  // ✅ Covariant (out)
// dogProducer = animalProducer;  // ❌ Error

declare let animalConsumer: Consumer<Animal>;
declare let dogConsumer: Consumer<Dog>;

dogConsumer = animalConsumer;  // ✅ Contravariant (in)
// animalConsumer = dogConsumer;  // ❌ Error

// Step 4: Conflict detection at declaration site.
// interface BadProducer<out T> {
//   produce(): T;
//   consume(value: T): void;  // ❌ Error: 'out' conflicts with usage as both source and sink.
// }
```

**Expected Output:** No runtime output (compile-time behavior only). The valid assignments compile; the invalid ones produce compile errors.

**Why This Output Occurs:** Explicit variance annotations (`out T`, `in T`, `in out T`) enforce the intended variance at the declaration site. If the usage conflicts with the annotation, the compiler errors at the declaration rather than at the consumer, making errors easier to diagnose.

### Real-World Cases

**Case 1: Event Handler Systems**
Event handlers use contravariance: a handler for `Animal` events can handle `Dog` events.

**Case 2: Immutable Collections**
Read-only collections use covariance: `ReadonlyArray<Dog>` is assignable to `ReadonlyArray<Animal>`.

**Case 3: Mutable Collections**
Mutable collections are invariant: `Array<Dog>` is not assignable to `Array<Animal>` (though TypeScript allows it unsoundly).

**Case 4: Function Composition**
Function composition libraries use covariance and contravariance to type composed functions correctly.

**Case 5: Redux Selectors**
Redux selectors use covariance for their return types, ensuring that a selector for `Animal` state can be used where a selector for `Dog` state is expected (under certain conditions).


## 3. Variance Annotations (`in`, `out`, `in out`) for Strict Generic Type Checking

### Definitions

**Core Definition**
Variance annotations are explicit modifiers (`in`, `out`, `in out`) on generic type parameters that declare the intended variance of the type parameter. Introduced in TypeScript 4.7, they allow authors to state whether a type parameter is contravariant (`in`), covariant (`out`), or invariant (`in out`), and the compiler verifies that the usage is consistent.

**Technical Definition**
Variance annotations are written before the type parameter name: `<out T>`, `<in T>`, or `<in out T>`. The `out` modifier declares covariance (the parameter is produced), `in` declares contravariance (the parameter is consumed), and `in out` declares invariance (the parameter is both produced and consumed). If the actual usage of the type parameter within the generic type violates the declared variance, the compiler produces an error at the declaration site. This shifts variance errors from consumer call sites (where they are hard to diagnose) to declaration sites (where the author can fix them). Variance annotations are purely a compile-time construct and have no runtime effect.

**Beginner-Friendly Explanation**
Variance annotations let you tell TypeScript what kind of variance a type parameter should have. If you write `<out T>`, you're saying "T is only produced (returned), so it's covariant." If you write `<in T>`, you're saying "T is only consumed (passed as a parameter), so it's contravariant." And `<in out T>` means "T is both produced and consumed, so it's invariant." TypeScript checks that your actual usage matches your annotation. If it doesn't, you get an error at the declaration, which is much easier to fix than an error at every use site. This is especially useful in complex generic interfaces where variance inference can be confusing.

### Purposes

- To declare the intended variance of a type parameter explicitly.
- To catch variance errors at the declaration site instead of the consumer site.
- To improve code readability and documentation of generic interfaces.
- To force the compiler to enforce a specific variance direction.
- To enable faster type checking by avoiding variance inference in complex cases.

### Syntax Rules and Structure

**General Syntax: Covariant Annotation**

```typescript
interface Producer<out T> {
  produce(): T;
}
```

**Component Breakdown**
- `<out T>`: `T` is covariant (produced).
- Usage: `T` must appear only in return positions.

**General Syntax: Contravariant Annotation**

```typescript
interface Consumer<in T> {
  consume(value: T): void;
}
```

**Component Breakdown**
- `<in T>`: `T` is contravariant (consumed).
- Usage: `T` must appear only in parameter positions.

**General Syntax: Invariant Annotation**

```typescript
interface State<in out T> {
  get(): T;
  set(value: T): void;
}
```

**Component Breakdown**
- `<in out T>`: `T` is invariant (both produced and consumed).
- Usage: `T` appears in both read and write positions.

**Syntax Rules**

- Annotations are placed before the type parameter name.
- `out T`: Covariant; `T` used only in return positions.
- `in T`: Contravariant; `T` used only in parameter positions.
- `in out T`: Invariant; `T` used in both positions.
- Annotations are checked at the declaration site.
- Conflicts between annotation and usage produce compile errors.
- Annotations apply to interfaces, type aliases, and classes (not generic functions).
- TypeScript 4.7+ is required.

**Constraints and Limitations**

- Variance annotations are not available for generic functions (only interfaces, type aliases, and classes).
- Using `out` on a parameter used in both positions produces an error.
- Using `in` on a parameter used in return positions produces an error.
- Annotations can be overly restrictive if usage patterns change.
- Inference is usually correct; annotations are primarily for documentation and error localization.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Covariant and Contravariant Annotations

```typescript
// Step 1: Covariant annotation (out).
interface Producer<out T> {
  produce(): T;
}

class Animal { name = ""; }
class Dog extends Animal { breed = ""; }

declare let animalProducer: Producer<Animal>;
declare let dogProducer: Producer<Dog>;

animalProducer = dogProducer;  // ✅ Dog is an Animal
// dogProducer = animalProducer;  // ❌ Error: Animal may not be a Dog

console.log("Covariance works.");

// Step 2: Contravariant annotation (in).
interface Consumer<in T> {
  consume(value: T): void;
}

declare let animalConsumer: Consumer<Animal>;
declare let dogConsumer: Consumer<Dog>;

dogConsumer = animalConsumer;  // ✅ Animal handler can handle Dogs
// animalConsumer = dogConsumer;  // ❌ Error: Dog handler may not handle Animals

console.log("Contravariance works.");

// Step 3: Conflict detection.
// interface BadProducer<out T> {
//   produce(): T;
//   consume(value: T): void;  // ❌ Error: 'out' conflicts with usage as both source and sink.
// }

// Step 4: Invariant annotation.
interface State<in out T> {
  get(): T;
  set(value: T): void;
}

declare let animalState: State<Animal>;
declare let dogState: State<Dog>;

// animalState = dogState;  // ❌ Error: Dog may not be an Animal
// dogState = animalState;  // ❌ Error: Animal may not be a Dog

console.log("Invariance enforced.");
```

**Expected Output:**
```
Covariance works.
Contravariance works.
Invariance enforced.
```

**Why This Output Occurs:** The `out T` annotation enforces covariance, allowing `Producer<Dog>` to be assigned to `Producer<Animal>` but not the reverse. The `in T` annotation enforces contravariance, allowing `Consumer<Animal>` to be assigned to `Consumer<Dog>` but not the reverse. The `in out T` annotation enforces invariance, preventing both directions.

#### Example 2: Production-Grade Handler with Variance Annotation

```typescript
// Step 1: Define a handler interface with contravariant annotation.
interface Handler<in T> {
  handle(value: T): void;
  describe(): string;  // No T — fine.
}

// Step 2: Create handlers for a class hierarchy.
class Animal { name = ""; }
class Dog extends Animal { breed = ""; }

const animalHandler: Handler<Animal> = {
  handle(value) { console.log(`Handling animal: ${value.name}`); },
  describe() { return "Animal handler"; },
};

const dogHandler: Handler<Dog> = {
  handle(value) { console.log(`Handling dog: ${value.name} (${value.breed})`); },
  describe() { return "Dog handler"; },
};

// Step 3: Contravariance allows assigning animal handler to dog handler.
const handler: Handler<Dog> = animalHandler;  // ✅
handler.handle({ name: "Rex", breed: "Lab" });  // "Handling animal: Rex"

// Step 4: The reverse is not allowed.
// const badHandler: Handler<Animal> = dogHandler;  // ❌ Error

// Step 5: Using property syntax instead of method syntax for strict checking.
interface StrictHandler<in T> {
  handle: (value: T) => void;  // Property syntax — contravariant under strictFunctionTypes
}
```

**Expected Output:**
```
Handling animal: Rex
```

**Why This Output Occurs:** The `in T` annotation makes `Handler` contravariant in `T`. `Handler<Animal>` can be assigned to `Handler<Dog>` because an Animal handler can handle Dogs. The reverse assignment fails because a Dog handler cannot handle all Animals.

### Real-World Cases

**Case 1: Event Emitter Libraries**
Event emitter libraries use variance annotations to declare that event handlers are contravariant in the event type.

**Case 2: Redux Reducers**
Redux reducers use variance annotations to declare state covariance and action contravariance.

**Case 3: React Component Props**
React component prop types use variance annotations to declare how props relate to component subtypes.

**Case 4: Parser Combinators**
Parser combinator libraries use variance annotations to declare how parsers relate to their input and output types.

**Case 5: Dependency Injection Containers**
DI containers use variance annotations to declare how service types relate to their implementations.


## 4. Generic Callbacks, Higher-Order Parameter Mapping, and Contextual Typing

### Definitions

**Core Definition**
Generic callbacks are functions passed as arguments to generic functions, where the callback's parameter types are inferred from the generic function's type signature. Higher-order parameter mapping uses generic types to map between parameter lists and callback signatures, enabling type-safe event systems, middleware, and functional composition.

**Technical Definition**
When a generic function accepts a callback, TypeScript uses contextual typing to infer the callback's parameter types from the expected function type. For example, in `function map<T, U>(array: T[], fn: (item: T) => U): U[]`, the callback `fn` receives a parameter of type `T` inferred from the array. Higher-order parameter mapping extends this by using mapped types and indexed access to map between a map of callback types and their parameter lists. The pattern `Parameters<CallbackMap[K]>` extracts the parameter list for a specific callback, enabling type-safe invocation of callbacks with the correct arguments. TypeScript's inference engine performs contextual typing in two phases: first inferring type parameters from non-context-sensitive arguments, then using those inferences to type context-sensitive arguments.

**Beginner-Friendly Explanation**
A generic callback is a function you pass to a generic function, where TypeScript figures out the callback's parameter types from the generic context. For example, if you pass a callback to `Array.map()`, TypeScript knows the callback receives the array's element type. Higher-order parameter mapping goes further: you can define a map of callback types (like an event map) and use generics to ensure that when you call a callback, you pass the right arguments. This is how type-safe event emitters and middleware systems work. TypeScript uses "contextual typing" to infer the callback's parameter types from how it's used.

### Purposes

- To enable type-safe callbacks in generic functions and methods.
- To infer callback parameter types from the generic context.
- To build type-safe event emitters and middleware systems.
- To map between callback maps and their parameter lists using `Parameters`.
- To enable functional composition and higher-order functions with type safety.

### Syntax Rules and Structure

**General Syntax: Generic Callback**

```typescript
function map<T, U>(array: T[], fn: (item: T) => U): U[] {
  return array.map(fn);
}

const lengths = map(["a", "bb", "ccc"], (s) => s.length);
// s is string (inferred from T), U is number.
```

**Component Breakdown**
- `fn: (item: T) => U`: The callback type using type parameters.
- The callback's parameter is contextually typed as `T`.

**General Syntax: Higher-Order Parameter Mapping**

```typescript
type CallbackMap = {
  add: (a: number, b: number) => number;
  greet: (name: string) => string;
};

function invoke<K extends keyof CallbackMap>(
  callbacks: CallbackMap,
  key: K,
  ...args: Parameters<CallbackMap[K]>
): ReturnType<CallbackMap[K]> {
  return callbacks[key](...args) as ReturnType<CallbackMap[K]>;
}
```

**Component Breakdown**
- `Parameters<CallbackMap[K]>`: Extracts the parameter list for the callback at key `K`.
- `ReturnType<CallbackMap[K]>`: Extracts the return type.

**General Syntax: Contextual Typing in Generic Functions**

```typescript
function createLogger<T extends string>(
  prefix: T,
  logFn: (message: `${T}: ${string}`) => void
): void {
  logFn(`${prefix}: hello`);
}

createLogger("app", (message) => console.log(message));
// message is "app: hello" (literal type)
```

**Component Breakdown**
- The callback's parameter type is contextually typed by the template literal type.

**Syntax Rules**

- Callback parameter types are inferred from the generic context.
- `Parameters<F>` extracts the parameter list of function type `F`.
- `ReturnType<F>` extracts the return type of function type `F`.
- Higher-order mapping uses indexed access (`CallbackMap[K]`) with `Parameters` and `ReturnType`.
- Contextual typing works when the callback is passed inline.
- Explicit annotations on callback parameters may be needed in complex cases.

**Constraints and Limitations**

- TypeScript cannot always follow correlations between generic type parameters in complex callback scenarios.
- Contextual typing may fail with object literals containing unannotated parameters.
- Type parameter inference in deep recursive or circular cases may produce `unknown`.
- The `Parameters` utility type is implemented as a conditional type, which may be deferred for generic types.
- Explicit annotations or type assertions may be needed as workarounds.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Generic Callbacks with Contextual Typing

```typescript
// Step 1: Define a generic function that accepts a callback.
function transform<T, U>(items: T[], fn: (item: T) => U): U[] {
  return items.map(fn);
}

// Step 2: Call with an inline callback — parameter is contextually typed.
const numbers = [1, 2, 3, 4, 5];

const doubled = transform(numbers, (n) => n * 2);
// n is number (inferred from T), return type is number[]

console.log(doubled);  // [2, 4, 6, 8, 10]

// Step 3: The callback can return a different type.
const strings = transform(numbers, (n) => `Item ${n}`);
// n is number, return type is string[]

console.log(strings);  // ["Item 1", "Item 2", "Item 3", "Item 4", "Item 5"]

// Step 4: Type safety is preserved.
const result: string[] = transform(numbers, (n) => n.toString());
console.log(result);  // ["1", "2", "3", "4", "5"]

// Step 5: Complex contextual typing with template literals.
function createMessage<T extends string>(
  prefix: T,
  formatter: (message: `${T}: ${string}`) => void
): void {
  formatter(`${prefix}: hello`);
}

createMessage("app", (message) => {
  // message is "app: hello"
  console.log(message);
});
```

**Expected Output:**
```
[ 2, 4, 6, 8, 10 ]
[ 'Item 1', 'Item 2', 'Item 3', 'Item 4', 'Item 5' ]
[ '1', '2', '3', '4', '5' ]
app: hello
```

**Why This Output Occurs:** The `transform` function uses contextual typing to infer the callback parameter type `T` from the array. The `createMessage` function uses a template literal type to contextually type the callback parameter. TypeScript infers the parameter types automatically, enabling type-safe callbacks without explicit annotations.

#### Example 2: Higher-Order Parameter Mapping

```typescript
// Step 1: Define a callback map.
type CallbackMap = {
  add: (a: number, b: number) => number;
  greet: (name: string) => string;
  toggle: (value: boolean) => boolean;
};

// Step 2: Define a function that invokes a callback by key.
function invoke<K extends keyof CallbackMap>(
  callbacks: CallbackMap,
  key: K,
  ...args: Parameters<CallbackMap[K]>
): ReturnType<CallbackMap[K]> {
  const fn = callbacks[key];
  return fn(...args) as ReturnType<CallbackMap[K]>;
}

// Step 3: Create a callback map instance.
const callbacks: CallbackMap = {
  add: (a, b) => a + b,
  greet: (name) => `Hello, ${name}`,
  toggle: (value) => !value,
};

// Step 4: Invoke with correct arguments — TypeScript enforces types.
console.log(invoke(callbacks, "add", 2, 3));        // 5
console.log(invoke(callbacks, "greet", "Alice"));   // "Hello, Alice"
console.log(invoke(callbacks, "toggle", true));     // false

// Step 5: Invalid arguments are compile errors.
// invoke(callbacks, "add", "2", 3);       // ❌ Error: string not assignable to number
// invoke(callbacks, "greet", 42);         // ❌ Error: number not assignable to string
// invoke(callbacks, "toggle");            // ❌ Error: expected 1 argument

// Step 6: The return type is correctly inferred.
const addResult: number = invoke(callbacks, "add", 1, 2);
const greetResult: string = invoke(callbacks, "greet", "Bob");
console.log(addResult);    // 3
console.log(greetResult);  // "Hello, Bob"
```

**Expected Output:**
```
5
Hello, Alice
false
3
Hello, Bob
```

**Why This Output Occurs:** The `invoke` function uses `Parameters<CallbackMap[K]>` to type the rest arguments and `ReturnType<CallbackMap[K]>` to type the return value. The `K extends keyof CallbackMap` constraint ensures the key is valid. TypeScript enforces that the arguments match the callback's parameter types and that the return type is correct.

### Real-World Cases

**Case 1: Event Emitters**
Type-safe event emitters use higher-order parameter mapping to map event names to their callback parameter types.

**Case 2: Middleware Systems**
Express-style middleware uses generic callbacks to type `req`, `res`, and `next` parameters contextually.

**Case 3: Redux Middleware**
Redux middleware uses generic callbacks to type actions and the `next` dispatch function.

**Case 4: Form Libraries**
Form libraries use generic callbacks to type field validators and form submission handlers.

**Case 5: Testing Frameworks**
Testing frameworks use generic callbacks to type test bodies and assertion functions.

**Case 6: Array Methods**
`Array.map`, `Array.filter`, `Array.reduce`, and `Array.forEach` all use generic callbacks with contextual typing.


## 5. Generic Factories and Constructor Signatures (`new (...args: any[]) => T`)

### Definitions

**Core Definition**
A generic factory is a function that creates instances of a generic type by accepting a constructor reference and constructor arguments, returning a properly typed instance. The constructor signature `new (...args: any[]) => T` describes any class whose constructor produces an instance of type `T`.

**Technical Definition**
The constructor signature `new (...args: any[]) => T` is a call signature with the `new` keyword, describing a constructable type. The `Constructor<T>` type alias (`type Constructor<T = {}> = new (...args: any[]) => T`) is the standard way to type constructor references. Generic factories accept such a constructor and use `ConstructorParameters<T>` to type the constructor arguments and `InstanceType<T>` to type the returned instance. The pattern enables type-safe instantiation of arbitrary classes, dependency injection, and plugin systems. Generic factories are distinct from static factory methods in that they are standalone functions or methods that operate on any constructable type.

**Beginner-Friendly Explanation**
A generic factory is a function that can create objects of any type. You pass it a class (the constructor), and it returns an instance of that class. The constructor signature `new (...args: any[]) => T` describes "a class that creates T instances." The `Constructor<T>` type alias is the standard way to type these constructor references. Generic factories are useful for dependency injection, plugin systems, and any situation where you need to create objects dynamically but still want type safety. TypeScript's `ConstructorParameters<T>` and `InstanceType<T>` utility types make this pattern work.

### Purposes

- To create instances of arbitrary classes with type safety.
- To enable dependency injection and plugin systems.
- To type factory functions that accept constructor references.
- To use `ConstructorParameters<T>` and `InstanceType<T>` for precise typing.
- To support mixin and decorator patterns that operate on classes.

### Syntax Rules and Structure

**General Syntax: Constructor Signature**

```typescript
type Constructor<T = {}> = new (...args: any[]) => T;
```

**Component Breakdown**
- `new (...args: any[]) => T`: Describes a constructable type.
- `T`: The instance type produced by the constructor.

**General Syntax: Generic Factory Function**

```typescript
function createInstance<T>(ctor: Constructor<T>): T {
  return new ctor();
}
```

**Component Breakdown**
- `ctor: Constructor<T>`: The constructor parameter.
- `): T`: The return type is the instance type.

**General Syntax: Generic Factory with Constructor Parameters**

```typescript
function createWithArgs<T extends Constructor>(
  ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new ctor(...args) as InstanceType<T>;
}
```

**Component Breakdown**
- `T extends Constructor`: The type parameter constrained to a constructor.
- `ConstructorParameters<T>`: Extracts the constructor's parameter list.
- `InstanceType<T>`: Extracts the instance type.

**General Syntax: Factory Type Alias**

```typescript
type Factory<T extends new (...args: any[]) => any> =
  (...args: ConstructorParameters<T>) => InstanceType<T>;
```

**Component Breakdown**
- `Factory<T>`: A type alias for a factory function that creates instances of `T`.

**Syntax Rules**

- The constructor signature uses `new (...args: any[]) => T`.
- The `Constructor<T>` type alias is the standard naming convention.
- `ConstructorParameters<T>` extracts the constructor's parameter list.
- `InstanceType<T>` extracts the instance type.
- Generic factories can be functions, methods, or classes.
- The constraint `T extends new (...args: any[]) => any` is used for constructor parameters.
- Abstract constructors use `abstract new (...args: any[]) => T`.

**Constraints and Limitations**

- The constructor signature does not enforce specific parameter types unless constrained.
- Use tuple types to specify exact constructor parameters when needed.
- Abstract classes require `abstract new` in the constraint.
- The `InstanceType<T>` utility requires `T` to be a constructor type.
- Type inference may need explicit type arguments in complex cases.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Generic Factory

```typescript
// Step 1: Define a constructor type alias.
type Constructor<T = {}> = new (...args: any[]) => T;

// Step 2: Define a generic factory function.
function createInstance<T>(ctor: Constructor<T>): T {
  return new ctor();
}

// Step 3: Define classes.
class User {
  constructor(public name: string = "Anonymous") {}
}

class Product {
  constructor(public title: string = "Untitled") {}
}

// Step 4: Create instances with the factory.
const user = createInstance(User);
console.log(user.name);  // "Anonymous"

const product = createInstance(Product);
console.log(product.title);  // "Untitled"

// Step 5: Type safety is preserved.
const user2: User = createInstance(User);      // ✅
const product2: Product = createInstance(Product); // ✅

// Step 6: Constructor parameters are not passed (factory doesn't accept them).
// createInstance(User, "Alice");  // ❌ Error: expected 1 argument.
```

**Expected Output:**
```
Anonymous
Untitled
```

**Why This Output Occurs:** The `createInstance` function accepts a `Constructor<T>` and returns `new ctor()`. The `Constructor<T>` type describes a class with a no-argument constructor. TypeScript infers `T` from the constructor argument, preserving the instance type.

#### Example 2: Factory with Constructor Parameters

```typescript
// Step 1: Define a constructor type.
type Constructor<T = {}> = new (...args: any[]) => T;

// Step 2: Define a factory with constructor parameters.
function createWithArgs<T extends Constructor>(
  ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new ctor(...args) as InstanceType<T>;
}

// Step 3: Define classes with constructor parameters.
class User {
  constructor(public name: string, public age: number) {}
}

class Product {
  constructor(public title: string, public price: number) {}
}

// Step 4: Create instances with arguments.
const user = createWithArgs(User, "Alice", 30);
console.log(user.name);  // "Alice"
console.log(user.age);   // 30

const product = createWithArgs(Product, "Laptop", 999);
console.log(product.title);  // "Laptop"
console.log(product.price);  // 999

// Step 5: Type safety is preserved.
const user2: User = createWithArgs(User, "Bob", 25);  // ✅

// Step 6: Invalid arguments are compile errors.
// createWithArgs(User, "Alice");         // ❌ Error: expected 2 arguments, got 1
// createWithArgs(User, "Alice", "30");   // ❌ Error: string not assignable to number

// Step 7: Factory type alias.
type Factory<T extends Constructor> = (
  ...args: ConstructorParameters<T>
) => InstanceType<T>;

const userFactory: Factory<typeof User> = (name, age) => new User(name, age);
const newUser = userFactory("Charlie", 35);
console.log(newUser.name);  // "Charlie"
```

**Expected Output:**
```
Alice
30
Laptop
999
Charlie
```

**Why This Output Occurs:** The `createWithArgs` function uses `ConstructorParameters<T>` to type the rest parameters and `InstanceType<T>` to type the return value. The `Factory<T>` type alias provides a reusable factory type. TypeScript enforces that the arguments match the constructor's parameter types and count.

### Real-World Cases

**Case 1: Dependency Injection Containers**
DI containers use generic factories to instantiate services with their dependencies, ensuring type safety across the container.

**Case 2: Plugin Systems**
Plugin systems use generic factories to instantiate plugins, each with its own constructor signature, while maintaining type safety.

**Case 3: ORM Entity Factories**
ORMs use generic factories to create entity instances from database rows, typing both the constructor and the resulting entity.

**Case 4: Testing Utilities**
Test utilities use generic factories to create test doubles (mocks, stubs) from real classes, preserving the class's interface.

**Case 5: React Component Factories**
React component factories use generic factories to create higher-order components that wrap any component type.

**Case 6: Mixin Patterns**
Mixin functions use constructor signatures to accept and return classes, enabling type-safe mixin composition.

---

## References

- TypeScript Handbook: Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook: Type Compatibility — https://www.typescriptlang.org/docs/handbook/type-compatibility.html
- TypeScript 2.1 Release Notes (`keyof` and Lookup Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html
- TypeScript 2.6 Release Notes (Strict function types, variance) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-6.html
- TypeScript 4.7 Release Notes (Optional Variance Annotations) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-7.html
- TypeScript 2.4 Release Notes (Improved inference for generics) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-4.html
- TypeScript 2.3 Release Notes (Generic Parameter Defaults) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-3.html
- TypeScript Documentation: Indexed Access Types — https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- TypeScript Documentation: Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript Documentation: Utility Types (`Parameters`, `ReturnType`, `InstanceType`, `ConstructorParameters`) — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Playground: Generic Parameter Defaults — https://www.typescriptlang.org/play/typescript/generics.ts.html
- TypeScript Playground: `keyof` and Lookup Types — https://www.typescriptlang.org/play/typescript/primitives/keyof-types.ts.html
- Stack Overflow: Difference between Variance, Covariance, Contravariance, Bivariance and Invariance — https://stackoverflow.com/questions/66410115
- Stack Overflow: Generic factory parameters in Typescript — https://stackoverflow.com/questions/42804182
- TypeScript Issue #63545: Missing contextual typing for object-literal callback parameter — https://github.com/microsoft/TypeScript/issues/63545
- TypeScript PR #47109: Indexed access types and mapped types for generic operations — https://github.com/microsoft/TypeScript/pull/47109
- TypeScript Issue #30581: Correlated union types — https://github.com/microsoft/TypeScript/issues/30581
- TypeScript Issue #47599: Missing contextual typing for callback parameters — https://github.com/microsoft/TypeScript/issues/47599
- TypeScript Deep Dive: Generics — https://basarat.gitbook.io/typescript/type-system/generics
- Total TypeScript: Generic Constraints — https://www.totaltypescript.com/workshops/typescript-pro-essentials/the-utils-folder/type-parameter-constraints-with-generic-functions/solution
- TypeScript ESLint: no-unnecessary-type-constraint — https://typescript-eslint.io/rules/no-unnecessary-type-constraint/