# TypeScript Generic Constraints: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Generic constraints in TypeScript are a mechanism for restricting the set of types that can be used as type arguments for a generic type parameter. By using the `extends` keyword, you declare that a type parameter must satisfy a minimum structural requirement—such as having certain properties, being assignable to a union, or matching a specific shape.

**Technical Definition**
A generic constraint is expressed using the `extends` keyword after a type parameter declaration: `<T extends Constraint>`. The constraint limits the possible types that `T` can be instantiated with to those assignable to `Constraint`. Inside the generic body, `T` is treated as having all the members of `Constraint`, enabling safe access to those members. Constraints are checked structurally, so any type with the required members satisfies the constraint regardless of its declaration site. Constraints can themselves be generic (e.g., `K extends keyof T`), can be union types, primitive types, or complex object types, and can be combined with conditional types for advanced type-level logic.

**Beginner-Friendly Explanation**
A generic constraint is a way to say "this type parameter must be at least this much." If you write `function logLength<T extends { length: number }>(value: T)`, you're telling TypeScript that `T` must have a `length` property that's a number. Now you can safely access `value.length` inside the function. Without the constraint, TypeScript wouldn't know that `T` has a `length` property, so it would give you an error. Constraints let you use generic code while still accessing specific properties—you get the flexibility of generics with the safety of knowing what's available.

### Key Characteristics

- **Structural compliance**: Constraints are checked structurally; any type with the required members satisfies the constraint.
- **Safe member access**: Inside the generic body, the type parameter is treated as having all constraint members.
- **Composability**: Constraints can be union types, intersections, primitives, or other generic types.
- **Key-based constraints**: `K extends keyof T` restricts keys to valid property names.
- **Conditional constraints**: Constraints can be combined with conditional types for type-level logic.
- **Runtime erasure**: Constraints are erased at compile time; they exist only in the type system.

### Prerequisites

- Basic knowledge of TypeScript generics
- Familiarity with the `extends` keyword in inheritance
- Understanding of structural typing
- Familiarity with the `keyof` operator and indexed access types (`T[K]`)

### Related Programming Areas

- **Type Theory**: Bounded quantification, parametric polymorphism
- **Type-Level Programming**: Conditional types, mapped types, `infer`
- **Structural Typing**: Duck typing, structural subtyping
- **Library Design**: Reusable APIs with type-safe constraints
- **Utility Types**: `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>`

### Core Concepts / Features

1. The `extends` Keyword for Minimum Structural Compliance
2. Constraining Type Parameters Against Primitives, Objects, and Unions
3. Key-Based Constraints Using `extends keyof` Patterns
4. Structural Constraints and Duck-Typing Enforcement
5. Conditional Constraints Within Generic Bodies


## 1. The `extends` Keyword for Minimum Structural Compliance

### Definitions

**Core Definition**
The `extends` keyword in a generic constraint declares that a type parameter must be assignable to (i.e., structurally compatible with) a specified constraint type. It establishes the minimum set of members that the type argument must provide.

**Technical Definition**
In a generic declaration `<T extends Constraint>`, the `extends` keyword constrains `T` to types that are assignable to `Constraint`. This is a structural check: any type whose members satisfy `Constraint`'s members is accepted. Inside the generic body, `T` is treated as a subtype of `Constraint`, so all of `Constraint`'s members are accessible on values of type `T`. The constraint can be any type expression—a primitive, an object type, a union, an intersection, or another generic type parameter. When a type argument is provided that does not satisfy the constraint, TypeScript produces a compile error with a descriptive message.

**Beginner-Friendly Explanation**
The `extends` keyword in generics is like a minimum requirement. When you write `<T extends { id: number }>`, you're saying "T must be an object with an `id` number property." Any type that has that `id` property works, even if it has lots of other properties. Inside the function, TypeScript knows that `T` has an `id`, so you can access `value.id`. If you try to pass a string (which doesn't have `id`), TypeScript gives you an error. The constraint guarantees that certain members are always available.

### Purposes

- To guarantee that certain members are available on the type parameter.
- To enable safe access to constraint members within the generic body.
- To provide meaningful compile errors when the constraint is not satisfied.
- To document the minimum requirements for a generic component.
- To enable type-safe generic algorithms and data structures.

### Syntax Rules and Structure

**General Syntax: Basic Constraint**

```typescript
function functionName<T extends Constraint>(param: T): ReturnType {
  // T is treated as a subtype of Constraint
}
```

**Component Breakdown**
- `<T extends Constraint>`: The type parameter with a constraint.
- Inside the body: `param` has all members of `Constraint`.

**General Syntax: Multiple Constraints via Intersection**

```typescript
function functionName<T extends A & B>(param: T): void {
  // T must satisfy both A and B
}
```

**Component Breakdown**
- `T extends A & B`: The type must satisfy both constraints.

**General Syntax: Constraint with Default**

```typescript
function functionName<T extends Constraint = DefaultType>(param: T): void {
  // T defaults to DefaultType if not inferred
}
```

**Component Breakdown**
- `<T extends Constraint = DefaultType>`: The default must satisfy the constraint.

**Syntax Rules**

- The `extends` keyword is used after the type parameter name.
- The constraint can be any type expression.
- Multiple constraints are combined with intersection (`&`).
- The constraint is checked structurally, not nominally.
- Inside the generic body, the type parameter is treated as a subtype of the constraint.
- A type parameter's default (if provided) must satisfy the constraint.
- Constraint violations produce compile errors at the call site or declaration.

**Constraints and Limitations**

- The constraint does not change the runtime representation of the type.
- Constraints can be recursive (`T extends U, U extends T` is an error).
- Very complex constraints can slow down the compiler.
- Constraints do not create nominal typing (structural compatibility is sufficient).
- The constraint must be a type, not a value.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Structural Constraint

```typescript
// Step 1: Define a constraint interface.
interface HasId {
  id: number;
}

// Step 2: Constrain a generic function.
function printId<T extends HasId>(value: T): void {
  // Inside the function, T is known to have an `id` property.
  console.log(`ID: ${value.id}`);
}

// Step 3: Call with a type that satisfies the constraint.
printId({ id: 1, name: "Alice" });  // "ID: 1"
printId({ id: 2 });                 // "ID: 2"

// Step 4: A class that structurally satisfies the constraint.
class User {
  constructor(public id: number, public name: string) {}
}

printId(new User(3, "Bob"));  // "ID: 3"

// Step 5: A type that does NOT satisfy the constraint.
// printId({ name: "Charlie" });  // ❌ Error: Property 'id' is missing.
// printId("hello");              // ❌ Error: string does not have 'id'.
```

**Expected Output:**
```
ID: 1
ID: 2
ID: 3
```

**Why This Output Occurs:** The constraint `T extends HasId` requires the type argument to have an `id: number` property. Objects with `id` (and optionally other properties) satisfy the constraint. Objects without `id` produce compile errors.

#### Example 2: Intersection Constraint

```typescript
// Step 1: Define two constraint interfaces.
interface HasId {
  id: number;
}

interface HasName {
  name: string;
}

// Step 2: Combine constraints with intersection.
function describeEntity<T extends HasId & HasName>(entity: T): string {
  // T has both `id` and `name`.
  return `Entity ${entity.id}: ${entity.name}`;
}

// Step 3: Call with a type that satisfies both constraints.
console.log(describeEntity({ id: 1, name: "Alice" }));
// "Entity 1: Alice"

console.log(describeEntity({ id: 2, name: "Bob", email: "bob@example.com" }));
// "Entity 2: Bob"

// Step 4: Missing either constraint produces an error.
// describeEntity({ id: 3 });           // ❌ Error: 'name' is missing.
// describeEntity({ name: "Charlie" }); // ❌ Error: 'id' is missing.
```

**Expected Output:**
```
Entity 1: Alice
Entity 2: Bob
```

**Why This Output Occurs:** The constraint `T extends HasId & HasName` requires the type argument to satisfy both interfaces. Objects with both `id` and `name` are accepted, including those with extra properties.

### Real-World Cases

**Case 1: Entity Repositories**
Generic repositories use `T extends { id: number }` to ensure all entities have an ID, enabling methods like `findById`.

**Case 2: Logging Utilities**
Logging functions use `T extends { errorCode: number; errorDescription: string }` to ensure logged errors have the expected shape.

**Case 3: React Components**
Generic React components use constraints like `T extends { id: string }` to ensure list items have keys.

**Case 4: Database Clients**
Database clients use `T extends Record<string, unknown>` to ensure query parameters are objects.


## 2. Constraining Type Parameters Against Primitives, Objects, and Unions

### Definitions

**Core Definition**
Generic constraints can be applied against primitive types (e.g., `string`, `number`), object types (interfaces, classes), and union types. This allows developers to restrict type parameters to specific categories of types while leveraging TypeScript's structural typing.

**Technical Definition**
A constraint can be any type expression: a primitive (`T extends string`), an object type (`T extends { id: number }`), a union (`T extends string | number`), or an intersection (`T extends A & B`). When the constraint is a primitive, the type argument must be assignable to that primitive. When the constraint is a union, the type argument must be assignable to at least one member of the union. When the constraint is an object type, the type argument must have all the required members. Constraints can also be combined with conditional types for advanced type-level logic.

**Beginner-Friendly Explanation**
You can constrain a generic to primitives, objects, or unions. If you write `<T extends string>`, only strings are allowed. If you write `<T extends string | number>`, strings and numbers are allowed. If you write `<T extends { id: number }>`, any object with an `id` is allowed. This lets you control exactly what types can be passed to your generic code. Primitives are useful for limiting to simple types; objects for requiring certain properties; unions for allowing a specific set of types.

### Purposes

- To restrict type parameters to primitive types when only primitives are supported.
- To allow a specific set of types via union constraints.
- To require certain properties via object constraints.
- To combine multiple constraints via intersection.
- To document the acceptable type categories for a generic component.

### Syntax Rules and Structure

**General Syntax: Primitive Constraint**

```typescript
function functionName<T extends string>(value: T): T {
  return value;
}
```

**Component Breakdown**
- `<T extends string>`: Only strings are allowed.

**General Syntax: Union Constraint**

```typescript
function functionName<T extends string | number>(value: T): T {
  return value;
}
```

**Component Breakdown**
- `<T extends string | number>`: Strings and numbers are allowed.

**General Syntax: Object Constraint**

```typescript
function functionName<T extends { id: number; name: string }>(value: T): T {
  return value;
}
```

**Component Breakdown**
- `<T extends { id: number; name: string }>`: Objects with `id` and `name` are allowed.

**General Syntax: Union of Object Types**

```typescript
type Shape = { kind: "circle"; radius: number } | { kind: "square"; side: number };

function functionName<T extends Shape>(shape: T): number {
  return 0;
}
```

**Component Breakdown**
- `<T extends Shape>`: Only circle or square shapes are allowed.

**Syntax Rules**

- The constraint can be a primitive, object, union, or intersection.
- For primitive constraints, the type argument must be assignable to the primitive.
- For union constraints, the type argument must be assignable to at least one member.
- For object constraints, the type argument must have all required members.
- Literal types are subtypes of their primitives and satisfy primitive constraints.
- The constraint does not prevent extra properties on object types.

**Constraints and Limitations**

- Primitive constraints do not narrow the type parameter to literal types (unless the argument is a literal).
- Union constraints allow any member of the union, not a subset.
- Object constraints require all specified members, even if they are optional in the type argument.
- Constraints are structural, so any type with the required members satisfies the constraint.
- Very specific object constraints reduce the reusability of generic components.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Primitive and Union Constraints

```typescript
// Step 1: Constrain to primitives.
function double<T extends number>(value: T): number {
  return value * 2;
}

console.log(double(21));       // 42
// double("hello");            // ❌ Error: string is not assignable to number.

// Step 2: Constrain to a union of primitives.
function stringify<T extends string | number | boolean>(value: T): string {
  return String(value);
}

console.log(stringify("hello"));  // "hello"
console.log(stringify(42));       // "42"
console.log(stringify(true));     // "true"
// stringify({});                 // ❌ Error: object is not assignable to the union.

// Step 3: Constrain to a union of literal types.
type Direction = "north" | "south" | "east" | "west";

function move<T extends Direction>(direction: T): string {
  return `Moving ${direction}`;
}

console.log(move("north"));  // "Moving north"
// move("up");               // ❌ Error: "up" is not assignable to Direction.

// Step 4: Constrain to an object type.
interface Point {
  x: number;
  y: number;
}

function translate<T extends Point>(point: T, dx: number, dy: number): T {
  return { ...point, x: point.x + dx, y: point.y + dy } as T;
}

const p = translate({ x: 0, y: 0, z: 0 }, 5, 10);
console.log(p);  // { x: 5, y: 10, z: 0 }
```

**Expected Output:**
```
42
hello
42
true
Moving north
{ x: 5, y: 10, z: 0 }
```

**Why This Output Occurs:** The `double` function only accepts numbers. The `stringify` function accepts strings, numbers, and booleans. The `move` function only accepts the four literal directions. The `translate` function accepts any object with `x` and `y` numbers, preserving extra properties via the generic type.

#### Example 2: Union of Object Types Constraint

```typescript
// Step 1: Define a union of object types.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

// Step 2: Constrain to the union.
function calculateArea<T extends Shape>(shape: T): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}

// Step 3: Call with valid shapes.
console.log(calculateArea({ kind: "circle", radius: 5 }).toFixed(2));
// "78.54"
console.log(calculateArea({ kind: "square", side: 4 }));
// 16
console.log(calculateArea({ kind: "rectangle", width: 3, height: 6 }));
// 18

// Step 4: Invalid shapes are rejected.
// calculateArea({ kind: "triangle", base: 3, height: 4 });
// ❌ Error: 'triangle' is not assignable to 'circle' | 'square' | 'rectangle'.
```

**Expected Output:**
```
78.54
16
18
```

**Why This Output Occurs:** The constraint `T extends Shape` restricts the type argument to the three shape variants. The switch statement narrows the union based on the discriminant, enabling safe access to variant-specific properties.

### Real-World Cases

**Case 1: Numeric Utilities**
Mathematical utilities use `T extends number` to accept only numbers, rejecting strings and objects.

**Case 2: String Manipulation**
String utilities use `T extends string` to accept only strings, leveraging string methods safely.

**Case 3: Event Systems**
Event systems use union constraints to restrict event types to a known set, enabling exhaustive handling.

**Case 4: Configuration Validators**
Config validators use object constraints to ensure configuration objects have required fields.


## 3. Key-Based Constraints Using `extends keyof` Patterns

### Definitions

**Core Definition**
Key-based constraints use the `keyof` operator combined with `extends` to restrict a type parameter to the keys (property names) of another type. The pattern `K extends keyof T` ensures that `K` is a valid property name of `T`, enabling type-safe property access.

**Technical Definition**
For a type `T`, `keyof T` is the union of all known public property names of `T`. When a type parameter is constrained as `K extends keyof T`, `K` can only be instantiated with one of `T`'s property names. This enables type-safe property access: `T[K]` is the indexed access type corresponding to the property `K`. The pattern is fundamental to utility types like `Pick<T, K>` and `Omit<T, K>`, and is used in generic functions that operate on object properties. The `extends` in `K extends keyof T` is a constraint (not inheritance), meaning `K` must be assignable to the union of keys.

**Beginner-Friendly Explanation**
The `keyof` operator gives you a union of all property names of a type. If you have `interface User { name: string; age: number }`, then `keyof User` is `"name" | "age"`. When you write `<K extends keyof T>`, you're saying "K must be one of the property names of T." This lets you write functions that work with any property of an object, while TypeScript ensures you only use valid property names. For example, a `getProperty` function can take an object and a key, and TypeScript will check that the key actually exists on the object.

### Purposes

- To restrict a type parameter to valid property names of another type.
- To enable type-safe property access via indexed access types (`T[K]`).
- To build generic utilities like `Pick`, `Omit`, and `getProperty`.
- To prevent typos in property names at compile time.
- To enable type-safe object manipulation in generic code.

### Syntax Rules and Structure

**General Syntax: Key-Based Constraint**

```typescript
function functionName<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

**Component Breakdown**
- `<T, K extends keyof T>`: `K` must be a property name of `T`.
- `): T[K]`: The return type is the type of the property at `key`.

**General Syntax: Key-Based Constraint with Default**

```typescript
function functionName<T, K extends keyof T = keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

**Component Breakdown**
- `K extends keyof T = keyof T`: Defaults to all keys.

**General Syntax: Multiple Keys**

```typescript
function functionName<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map((key) => obj[key]);
}
```

**Component Breakdown**
- `keys: K[]`: An array of valid keys.
- `): T[K][]`: An array of the corresponding property values.

**Syntax Rules**

- `keyof T` returns a union of property names.
- `K extends keyof T` restricts `K` to those names.
- `T[K]` is the indexed access type for property `K`.
- The constraint works with interfaces, type aliases, classes, and index signatures.
- `keyof T` includes optional properties.
- `keyof T` excludes private and protected members.
- For types with index signatures, `keyof T` includes `string | number`.

**Constraints and Limitations**

- `keyof` on a type without properties returns `never`.
- `keyof` on `any` returns `string | number | symbol`.
- `keyof` on a union distributes, producing the union of all keys (not the intersection).
- Index signatures affect `keyof` results (`string | number` for `[key: string]`).
- `K extends keyof T` does not narrow `T` itself.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `keyof` Constraint

```typescript
// Step 1: Define an interface.
interface Person {
  name: string;
  age: number;
  email: string;
}

// Step 2: Define a generic function with a key constraint.
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Step 3: Use the function with valid keys.
const person: Person = { name: "Alice", age: 30, email: "alice@example.com" };

const name = getProperty(person, "name");   // string
const age = getProperty(person, "age");     // number
const email = getProperty(person, "email"); // string

console.log(name);   // "Alice"
console.log(age);    // 30
console.log(email);  // "alice@example.com"

// Step 4: Invalid keys produce compile errors.
// getProperty(person, "gender");  // ❌ Error: "gender" is not assignable to "name" | "age" | "email".
// getProperty(person, "address"); // ❌ Error: "address" is not a key of Person.

// Step 5: Type safety is preserved.
const nameLength: number = getProperty(person, "name").length;  // ✅
const ageFixed: string = getProperty(person, "age").toFixed(2); // ✅
```

**Expected Output:**
```
Alice
30
alice@example.com
```

**Why This Output Occurs:** The constraint `K extends keyof T` restricts `key` to `"name" | "age" | "email"`. The return type `T[K]` is the specific type of the property, so `getProperty(person, "age")` returns `number`. Invalid keys are caught at compile time.

#### Example 2: `keyof` Constraint with Multiple Keys and Defaults

```typescript
// Step 1: Define an interface.
interface Config {
  apiUrl: string;
  timeout: number;
  retries: number;
}

// Step 2: Get multiple properties.
function getProperties<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map((key) => obj[key]);
}

const config: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
};

const values = getProperties(config, ["apiUrl", "timeout"]);
console.log(values);  // ["https://api.example.com", 5000]

// Step 3: Key constraint with default.
function getPropertyWithDefault<T, K extends keyof T = keyof T>(
  obj: T,
  key?: K
): T[K] | undefined {
  if (key === undefined) return undefined;
  return obj[key];
}

console.log(getPropertyWithDefault(config, "retries"));  // 3
console.log(getPropertyWithDefault(config));             // undefined

// Step 4: Constraint with index signatures.
interface StringMap {
  [key: string]: string;
}

function getMapValue<T extends StringMap, K extends keyof T>(map: T, key: K): string {
  return map[key];
}

const map: StringMap = { a: "hello", b: "world" };
console.log(getMapValue(map, "a"));  // "hello"
```

**Expected Output:**
```
[ 'https://api.example.com', 5000 ]
3
undefined
hello
```

**Why This Output Occurs:** The `getProperties` function accepts an array of valid keys and returns their values. The `getPropertyWithDefault` function defaults to all keys. The `getMapValue` function works with index signatures, where `keyof T` is `string | number`.

### Real-World Cases

**Case 1: Form Libraries**
Form libraries use `K extends keyof FormValues` to type field names and ensure type-safe access to form values.

**Case 2: State Management**
State management utilities use `keyof` constraints to type action creators and reducers that operate on state properties.

**Case 3: Configuration Access**
Configuration helpers use `keyof Config` to provide type-safe access to configuration values.

**Case 4: Data Transformation**
Data transformation utilities use `keyof` constraints to type functions that pick, omit, or transform object properties.

**Case 5: Database Query Builders**
Query builders use `keyof Entity` to type column names, preventing SQL injection and typos.


## 4. Structural Constraints and Duck-Typing Enforcement

### Definitions

**Core Definition**
Structural constraints enforce duck typing: a type satisfies a constraint if it has the required members, regardless of its name or declaration site. This is a fundamental characteristic of TypeScript's type system, where compatibility is determined by structure rather than nominal identity.

**Technical Definition**
TypeScript's type system is structurally typed: two types are compatible if their members are compatible. When a generic constraint is applied (`T extends Constraint`), any type whose members satisfy `Constraint`'s members is accepted. This means classes, interfaces, and object literals can all satisfy the same constraint without sharing a common ancestor. The exception is classes with `private` or `protected` members, which introduce nominal behavior—only types that inherit from the same declaration satisfy the constraint. Structural constraints enable duck typing: "if it walks like a duck and quacks like a duck, it's a duck."

**Beginner-Friendly Explanation**
Structural constraints mean TypeScript only cares about what a type *has*, not what it's *called*. If your constraint requires an `id` property and a `name` property, any object with those properties works—whether it's a class instance, an interface, or a plain object literal. You don't need to explicitly implement an interface. This is called "duck typing": if it looks like the constraint, it satisfies the constraint. The only exception is classes with `private` members, which require actual inheritance. This flexibility is one of TypeScript's most powerful features.

### Purposes

- To enable duck typing: any type with the required members satisfies the constraint.
- To allow unrelated classes and interfaces to satisfy the same constraint.
- To decouple generic components from specific type names and hierarchies.
- To work with plain objects, class instances, and interfaces interchangeably.
- To enforce minimum structural compliance without requiring explicit implementation.

### Syntax Rules and Structure

**General Syntax: Structural Constraint**

```typescript
interface HasId {
  id: number;
}

function functionName<T extends HasId>(value: T): T {
  return value;
}

// Any type with `id: number` satisfies the constraint.
functionName({ id: 1 });
functionName(new User(2));
functionName({ id: 3, name: "Alice" });
```

**Component Breakdown**
- `<T extends HasId>`: The structural constraint.
- Any type with an `id` number property is accepted.

**General Syntax: Nominal Exception with Private Members**

```typescript
class Base {
  private secret: string = "hidden";
}

class Derived extends Base {}

function functionName<T extends Base>(value: T): T {
  return value;
}

functionName(new Derived());  // ✅
// functionName({ secret: "hidden" });  // ❌ Error: private member mismatch.
```

**Component Breakdown**
- Private members make the constraint nominal.

**Syntax Rules**

- Structural constraints are checked by member compatibility, not name.
- Any type with the required members satisfies the constraint.
- Classes, interfaces, and object literals can all satisfy the same constraint.
- Private and protected members introduce nominal behavior.
- Extra properties on the type argument are allowed.
- The constraint does not require explicit `implements` or `extends`.

**Constraints and Limitations**

- Structural typing can allow unintended compatibility between similar shapes.
- Private and protected members break structural compatibility (nominal).
- Excess property checking applies to direct object literals but not variables.
- Structural constraints can make type errors less obvious.
- Duck typing does not provide nominal guarantees.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Structural Constraint with Different Types

```typescript
// Step 1: Define a structural constraint.
interface Serializable {
  serialize(): string;
}

// Step 2: Constrain a generic function.
function save<T extends Serializable>(entity: T): string {
  return entity.serialize();
}

// Step 3: An interface that structurally satisfies the constraint.
interface User {
  name: string;
  serialize(): string;
}

const user: User = {
  name: "Alice",
  serialize() { return JSON.stringify(this); },
};

console.log(save(user));  // '{"name":"Alice","serialize":...}'

// Step 4: A class that structurally satisfies the constraint.
class Product {
  constructor(public name: string, public price: number) {}
  serialize(): string { return JSON.stringify(this); }
}

console.log(save(new Product("Laptop", 999)));
// '{"name":"Laptop","price":999}'

// Step 5: A plain object that structurally satisfies the constraint.
console.log(save({ serialize: () => "custom" }));  // "custom"

// Step 6: A type without `serialize` is rejected.
// save({ name: "Charlie" });  // ❌ Error: 'serialize' is missing.
```

**Expected Output:**
```
{"name":"Alice","serialize":...}
{"name":"Laptop","price":999}
custom
```

**Why This Output Occurs:** The constraint `T extends Serializable` requires a `serialize` method. Any type with that method—interface, class, or plain object—satisfies the constraint. No explicit `implements` is needed because TypeScript is structurally typed.

#### Example 2: Nominal Behavior with Private Members

```typescript
// Step 1: Define a class with a private member.
class BaseEntity {
  private id: number;
  constructor(id: number) {
    this.id = id;
  }
  getId(): number {
    return this.id;
  }
}

// Step 2: Constrain a generic function.
function processEntity<T extends BaseEntity>(entity: T): number {
  return entity.getId();
}

// Step 3: A derived class satisfies the constraint.
class User extends BaseEntity {
  constructor(id: number, public name: string) {
    super(id);
  }
}

console.log(processEntity(new User(1, "Alice")));  // 1

// Step 4: A structurally similar class does NOT satisfy the constraint.
class FakeEntity {
  private id: number;
  constructor(id: number) {
    this.id = id;
  }
  getId(): number {
    return this.id;
  }
}

// processEntity(new FakeEntity(2));
// ❌ Error: Types have separate declarations of a private property 'id'.

// Step 5: A plain object does NOT satisfy the constraint.
// processEntity({ id: 3, getId: () => 3 });
// ❌ Error: Property 'getId' is private in type 'BaseEntity' but not in the object.
```

**Expected Output:**
```
1
```

**Why This Output Occurs:** The `private id` member in `BaseEntity` makes the constraint nominal. Only classes that inherit from `BaseEntity` (or share its private member declaration) satisfy the constraint. Structurally similar classes and plain objects are rejected because they have separate declarations of the private member.

### Real-World Cases

**Case 1: Plugin Systems**
Plugin systems use structural constraints to accept any plugin that implements the required methods, without requiring a common base class.

**Case 2: Dependency Injection**
DI containers use structural constraints to resolve dependencies based on the shape of the dependency, not its type name.

**Case 3: Testing Mocks**
Tests use structural constraints to accept mock objects that have the same shape as production dependencies, without instantiating real classes.

**Case 4: API Clients**
API clients use structural constraints to accept any object with the required HTTP methods, enabling different HTTP libraries.

**Case 5: Serialization**
Serialization utilities use structural constraints to accept any object with a `toJSON` method, matching JavaScript's JSON serialization protocol.


## 5. Conditional Constraints Within Generic Bodies

### Definitions

**Core Definition**
Conditional constraints combine generic constraints with conditional types to create type-level logic that varies based on whether a type parameter satisfies a condition. Inside a generic body, conditional types can inspect the type parameter and produce different types depending on the outcome.

**Technical Definition**
A conditional type has the form `T extends U ? X : Y`. When used within a generic body (as a type alias or return type), it evaluates based on the type argument provided. The `extends` in a conditional type is a type-level check: it asks "is `T` assignable to `U`?" (not "does `T` inherit from `U`"). Conditional types become distributive when the checked type is a naked type parameter: `T extends U ? X : Y` distributes over unions in `T`, applying the conditional to each member. The `infer` keyword enables type extraction within conditional types. Conditional constraints are commonly used for utility types, type-level validation, and function overload resolution.

**Beginner-Friendly Explanation**
A conditional type is like an "if" statement at the type level. You write `T extends U ? X : Y`, which means "if T is assignable to U, use type X; otherwise, use type Y." When combined with generics, this lets you create types that change based on what type parameter you pass in. For example, a `Result<T>` type might return `{ data: T }` if `T` is not an error, or `{ error: T }` if it is. Conditional types are powerful but complex—they're used in advanced type-level programming and utility types.

### Purposes

- To create type-level logic that varies based on the type argument.
- To implement utility types like `Exclude<T, U>`, `Extract<T, U>`, and `NonNullable<T>`.
- To enable type-safe function overloads and return type inference.
- To filter and transform unions in type-level operations.
- To build advanced generic APIs with conditional behavior.

### Syntax Rules and Structure

**General Syntax: Conditional Type**

```typescript
type ConditionalType<T> = T extends U ? X : Y;
```

**Component Breakdown**
- `T extends U ? X : Y`: If `T` is assignable to `U`, the type is `X`; otherwise, `Y`.

**General Syntax: Conditional Type with `infer`**

```typescript
type UnwrapArray<T> = T extends (infer U)[] ? U : T;
```

**Component Breakdown**
- `infer U`: Captures the element type of the array.

**General Syntax: Distributive Conditional Type**

```typescript
type ToArray<T> = T extends any ? T[] : never;
// ToArray<string | number> = string[] | number[]
```

**Component Breakdown**
- The conditional distributes over union members.

**General Syntax: Conditional Constraint in Generic Body**

```typescript
function functionName<T>(value: T): T extends string ? string : T {
  if (typeof value === "string") {
    return value.toUpperCase() as any;
  }
  return value as any;
}
```

**Component Breakdown**
- The return type uses a conditional type based on `T`.

**Syntax Rules**

- Conditional types use the form `T extends U ? X : Y`.
- The `extends` in conditional types is a type-level assignability check.
- Conditional types distribute over naked type parameters when the checked type is a union.
- `infer` declares a type variable within the true branch.
- Conditional types can be nested.
- Conditional types are commonly used in type aliases, not function bodies (due to runtime/type separation).

**Constraints and Limitations**

- Conditional types with generic parameters are often deferred until instantiation.
- Distributive conditional types can produce unexpected results with unions.
- `infer` can only be used within the true branch of a conditional type.
- Conditional types cannot be used to perform runtime logic.
- Very complex conditional types can slow down the compiler and produce unreadable errors.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Conditional Type

```typescript
// Step 1: Define a conditional type.
type IsString<T> = T extends string ? true : false;

// Step 2: Use it with different types.
type A = IsString<string>;   // true
type B = IsString<number>;   // false
type C = IsString<"hello">;  // true (literal extends string)

// Step 3: Conditional type with generic function.
function describeType<T>(value: T): IsString<T> extends true ? string : T {
  if (typeof value === "string") {
    return `String: ${value.toUpperCase()}` as any;
  }
  return value as any;
}

console.log(describeType("hello"));  // "String: HELLO"
console.log(describeType(42));       // 42

// Step 4: Distributive conditional type.
type ToArray<T> = T extends any ? T[] : never;

type StringOrNumberArray = ToArray<string | number>;
// string[] | number[] (distributed)

// Step 5: Utility type using conditional.
type NonNullable<T> = T extends null | undefined ? never : T;

type Defined = NonNullable<string | null | undefined>;  // string
const value: Defined = "hello";
console.log(value);  // "hello"
```

**Expected Output:**
```
String: HELLO
42
hello
```

**Why This Output Occurs:** The `IsString<T>` conditional type evaluates to `true` for strings and `false` otherwise. The `describeType` function uses this to vary its return type. The `ToArray` type distributes over unions. The `NonNullable` utility filters out `null` and `undefined`.

#### Example 2: Conditional Constraint for Type-Safe Property Access

```typescript
// Step 1: Define a conditional type for property access.
type PropType<T, K extends keyof T> = T[K] extends infer U ? U : never;

interface User {
  id: number;
  name: string;
  email: string;
}

// Step 2: Extract property types conditionally.
type UserId = PropType<User, "id">;     // number
type UserName = PropType<User, "name">; // string

// Step 3: Conditional type for function return.
type ReturnTypeOf<T> = T extends (...args: any[]) => infer R ? R : never;

function greet(name: string): string {
  return `Hello, ${name}`;
}

type GreetReturn = ReturnTypeOf<typeof greet>;  // string
const greeting: GreetReturn = "Hello";
console.log(greeting);  // "Hello"

// Step 4: Conditional constraint with union.
type ExtractStrings<T> = T extends string ? T : never;

type Mixed = "a" | 1 | "b" | 2;
type OnlyStrings = ExtractStrings<Mixed>;  // "a" | "b"

const s: OnlyStrings = "a";
console.log(s);  // "a"

// Step 5: Nested conditional types.
type DeepReadonly<T> = T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

interface Config {
  api: { url: string; timeout: number };
}

type ReadonlyConfig = DeepReadonly<Config>;
// All properties recursively readonly.
```

**Expected Output:**
```
Hello
a
```

**Why This Output Occurs:** The `PropType` conditional type extracts the type of a property. The `ReturnTypeOf` conditional type infers the return type of a function. The `ExtractStrings` conditional type filters a union to only string members. The `DeepReadonly` conditional type recursively makes all properties readonly.

### Real-World Cases

**Case 1: Utility Type Libraries**
Libraries like `type-fest` and `ts-essentials` use conditional types extensively for utility types.

**Case 2: API Response Typing**
API clients use conditional types to infer response types based on request parameters.

**Case 3: Form Validation**
Form libraries use conditional types to derive validation result types from form schemas.

**Case 4: State Management**
State management libraries use conditional types to type reducers and selectors based on state shape.

**Case 5: Type-Safe Event Emitters**
Event emitters use conditional types to map event names to payload types.

---

## References

- TypeScript Handbook: Generic Constraints — https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints
- TypeScript Handbook: Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Handbook: Keyof Type Operator — https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TypeScript Handbook: Indexed Access Types — https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Handbook: Type Compatibility — https://www.typescriptlang.org/docs/handbook/type-compatibility.html
- TypeScript 2.8 Release Notes (Conditional Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html
- TypeScript 2.1 Release Notes (keyof and Lookup Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html
- Stack Overflow: `extends keyof` and `in keyof` Meaning — https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/57337598/in-typescript-what-do-extends-keyof-and-in-keyof-mean
- Stack Overflow: Constraining a Generic Parameter to a Union Type — https://stackoverflow.com/questions/69035651/constraining-a-generic-parameter-to-be-a-union-type-in-typescript
- Total TypeScript: Generic Constraints — https://www.totaltypescript.com/workshops/typescript-pro-essentials/the-utils-folder/type-parameter-constraints-with-generic-functions/solution
- TypeScript ESLint: no-unnecessary-type-constraint — https://typescript-eslint.io/rules/no-unnecessary-type-constraint/
- Effective TypeScript: Item 14 — Use Type Operations and Generics to Avoid Repeating Yourself