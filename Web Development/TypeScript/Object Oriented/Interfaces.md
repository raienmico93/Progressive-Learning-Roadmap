# TypeScript Interfaces: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An interface in TypeScript is a named declaration that describes the shape of an object—the properties it must have, the methods it must implement, and the function or constructor signatures it must support. Interfaces are one of the primary tools for defining contracts in TypeScript and are purely a compile-time construct with no runtime representation.

**Technical Definition**
An interface is a TypeScript type declaration introduced with the `interface` keyword that defines a structural contract. Interfaces describe object types via property declarations, method signatures, function call signatures, construct signatures, and index signatures. They support optional (`?`) and readonly modifiers on properties, and can extend one or more other interfaces via the `extends` clause. A distinctive feature of interfaces is declaration merging: multiple declarations with the same name in the same scope are merged into a single interface, enabling augmentation of existing types. Interfaces do not support unions, intersections, or primitive aliases directly, and they cannot represent tuple or union types without workarounds.

**Beginner-Friendly Explanation**
An interface is a contract that describes what an object should look like. If you define an interface called `User` with `name` and `email` properties, any object you label as a `User` must have those properties. Interfaces are like a blueprint or a checklist: they tell TypeScript what to expect, and TypeScript checks that objects match. Interfaces can be extended (one interface can build on another), and they can be merged (if you declare the same interface twice, TypeScript combines them). Interfaces are most useful for describing objects and classes, but they can also describe functions and constructors.

### Key Characteristics

- **Structural contract**: Interfaces describe object shapes using structural typing.
- **Named and reusable**: Unlike anonymous object types, interfaces have names and can be referenced everywhere.
- **Extensible**: Interfaces can extend one or more other interfaces.
- **Mergeable**: Multiple declarations with the same name are merged (declaration merging).
- **Modifiers**: Properties can be optional (`?`) or readonly (`readonly`).
- **Multiple member kinds**: Properties, methods, function call signatures, construct signatures, and index signatures.
- **Compile-time only**: Erased during compilation; no runtime representation.

### Prerequisites

- Basic knowledge of JavaScript objects and functions
- Familiarity with TypeScript object types
- Understanding of TypeScript's type annotations and inference
- Familiarity with classes (for implementing interfaces)

### Related Programming Areas

- **Object-Oriented Programming**: Interfaces define contracts for classes
- **Structural Typing**: Interfaces are TypeScript's primary structural typing construct
- **Design Patterns**: Interfaces are fundamental to dependency injection, strategy pattern, and adapter pattern
- **API Design**: Interfaces define public API contracts
- **Module Augmentation**: Declaration merging enables extending existing types

### Core Concepts / Features

1. Interface Declaration Syntax
2. Interface Properties (Optional and Readonly)
3. Method Definitions vs Function Property Syntax
4. Function and Construct Signatures in Interfaces
5. Interface Extension (extends with Single/Multiple Interfaces)
6. Declaration Merging (Interface Merging Behavior)


## 1. Interface Declaration Syntax

### Definitions

**Core Definition**
Interface declaration syntax is the TypeScript syntax used to define a named interface using the `interface` keyword, an interface name, and a body of member declarations enclosed in braces. The declaration creates a named type that describes an object's structure.

**Technical Definition**
An interface declaration follows the grammar `interface Identifier TypeParameters? InterfaceExtendsClause? ObjectType`. The `Identifier` is the interface name, `TypeParameters` is an optional list of generic parameters, `InterfaceExtendsClause` is an optional list of interfaces to extend, and `ObjectType` is the body containing member declarations. Interface members can be properties, methods, call signatures, construct signatures, or index signatures. Interface names are in the type namespace and do not conflict with variable or class names in the value namespace (though interfaces can merge with classes and namespaces).

**Beginner-Friendly Explanation**
An interface declaration is how you write an interface. You start with the `interface` keyword, then the interface name (usually PascalCase), then a set of curly braces with the members inside. Each member is a property or method that objects of this interface must have. For example, `interface User { name: string; age: number }` says that a `User` has a `name` (string) and an `age` (number). The name goes in TypeScript's "type space"—it doesn't create a runtime variable.

### Purposes

- To define named contracts for object shapes that can be referenced throughout a codebase.
- To describe the structure of data passed between functions and modules.
- To define contracts that classes can implement.
- To serve as documentation for the expected shape of API data.
- To provide a foundation for declaration merging and module augmentation.

### Syntax Rules and Structure

**General Syntax: Basic Interface Declaration**

```typescript
interface InterfaceName {
  property1: Type1;
  property2: Type2;
  method1(param: Type): ReturnType;
}
```

**Component Breakdown**
- `interface`: The keyword introducing the interface declaration.
- `InterfaceName`: The name of the interface (PascalCase by convention).
- `{ ... }`: The interface body containing member declarations.
- `property1: Type1`: A property declaration.
- `method1(param: Type): ReturnType`: A method declaration.

**General Syntax: Generic Interface Declaration**

```typescript
interface InterfaceName<T, U = DefaultType> {
  property: T;
  method(input: U): T;
}
```

**Component Breakdown**
- `<T, U = DefaultType>`: Generic type parameters with an optional default.
- The interface can use `T` and `U` in its members.

**General Syntax: Interface with Extends Clause**

```typescript
interface ChildInterface extends ParentInterface1, ParentInterface2 {
  additionalProperty: Type;
}
```

**Component Breakdown**
- `extends ParentInterface1, ParentInterface2`: The extends clause listing parent interfaces.
- The child interface inherits all members from parents.

**Syntax Rules**

- The `interface` keyword is followed by the interface name.
- The interface body is enclosed in curly braces.
- Members are separated by semicolons, commas, or newlines.
- Property declarations use `name: Type` syntax.
- Method declarations use `name(params): ReturnType` syntax.
- Generic parameters are declared in angle brackets after the interface name.
- The `extends` clause follows the interface name (and generic parameters).
- Interfaces can be exported and imported using standard module syntax.
- Interfaces exist in the type namespace and do not generate runtime code.

**Constraints and Limitations**

- Interfaces cannot represent unions, intersections (directly), or primitive types.
- Interfaces cannot be extended by type aliases (though type aliases can intersect with interfaces).
- Interfaces are erased at runtime and cannot be checked with `instanceof`.
- Interfaces do not support computed property names with non-literal expressions in some contexts.
- Excess property checking applies to object literals assigned to interface types.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Interface Declaration

```typescript
// Step 1: Declare an interface with properties and a method.
interface User {
  id: number;
  name: string;
  email: string;
  greet(): string;
}

// Step 2: Create an object that conforms to the interface.
const alice: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  greet() {
    return `Hello, I'm ${this.name}`;
  },
};

// Step 3: Use the object with type safety.
console.log(alice.greet());    // "Hello, I'm Alice"
console.log(alice.email);       // "alice@example.com"

// Step 4: Missing properties cause errors.
// const bob: User = { id: 2, name: "Bob" };
// ❌ Error: Property 'email' is missing in type '{ id: number; name: string; }' but required in type 'User'.

// Step 5: Extra properties on object literals trigger excess property checking.
// const charlie: User = {
//   id: 3, name: "Charlie", email: "c@example.com", phone: "555-0123"
// };
// ❌ Error: Object literal may only specify known properties.
```

**Expected Output:**
```
Hello, I'm Alice
alice@example.com
```

**Why This Output Occurs:** The `User` interface requires `id`, `name`, `email`, and `greet`. The object `alice` satisfies all requirements, so it compiles. Missing properties and extra properties on direct object literals trigger compile errors.

#### Example 2: Generic Interface

```typescript
// Step 1: Declare a generic interface.
interface Container<T> {
  value: T;
  getValue(): T;
  map<U>(fn: (value: T) => U): Container<U>;
}

// Step 2: Create a Container implementation.
function createContainer<T>(value: T): Container<T> {
  return {
    value,
    getValue() {
      return value;
    },
    map<U>(fn: (value: T) => U): Container<U> {
      return createContainer(fn(value));
    },
  };
}

// Step 3: Use the container with different types.
const numberContainer = createContainer(42);
console.log(numberContainer.getValue());  // 42

const stringContainer = numberContainer.map((n) => n.toString());
console.log(stringContainer.getValue());  // "42"

const arrayContainer = numberContainer.map((n) => [n, n * 2]);
console.log(arrayContainer.getValue());   // [42, 84]
```

**Expected Output:**
```
42
42
[ 42, 84 ]
```

**Why This Output Occurs:** The generic `Container<T>` interface describes a box holding a value of type `T`. The `map` method transforms the container's contents, changing the type from `T` to `U`. The implementation respects the interface contract.

### Real-World Cases

**Case 1: API Data Contracts**
Interfaces define the shape of API responses, ensuring that consumers access only properties that exist and with the correct types.

**Case 2: React Props**
React components use interfaces to define their props, enabling type-safe JSX usage and autocompletion.

**Case 3: Domain Models**
Domain entities (User, Order, Product) are often defined as interfaces, providing a shared vocabulary across the codebase.

---

## 2. Interface Properties (Optional and Readonly)

### Definitions

**Core Definition**
Interface properties can be modified with the optional modifier (`?`) to indicate that a property may be absent, or the readonly modifier (`readonly`) to indicate that a property cannot be reassigned after creation. These modifiers provide fine-grained control over how interface properties behave.

**Technical Definition**
The optional modifier (`?`) transforms a property's type from `T` to `T | undefined` and allows the property to be omitted when creating an object of that interface type. Under `strictNullChecks`, accessing an optional property without narrowing produces a compile error. The readonly modifier prevents assignment to the property after initialization (shallow readonly). Neither modifier affects runtime behavior. With `exactOptionalPropertyTypes` (TypeScript 4.4+), optional properties cannot be assigned `undefined` explicitly unless the type includes `undefined`. Both modifiers can be combined (`readonly prop?: Type`).

**Beginner-Friendly Explanation**
Optional properties are ones that might not be there. If an interface has `email?: string`, an object can have an `email` or omit it entirely. Readonly properties are ones you can't change after creating the object. If `id` is `readonly`, you can read `user.id` but not write `user.id = 5`. These modifiers make interfaces more flexible (optional) and safer (readonly). You can use both on the same property: `readonly email?: string` means the property is optional and, if present, can't be changed.

### Purposes

- To model objects where some properties may be absent (optional).
- To prevent accidental mutation of properties that should remain constant (readonly).
- To enable configuration objects with optional settings.
- To document immutability intent for interface properties.
- To combine optionality and immutability for maximum flexibility with safety.

### Syntax Rules and Structure

**General Syntax: Optional Property**

```typescript
interface InterfaceName {
  requiredProperty: Type1;
  optionalProperty?: Type2;
}
```

**Component Breakdown**
- `optionalProperty?`: The `?` marks the property as optional.
- The property's type becomes `Type2 | undefined`.
- The property may be omitted when creating an object.

**General Syntax: Readonly Property**

```typescript
interface InterfaceName {
  readonly propertyName: Type;
}
```

**Component Breakdown**
- `readonly`: The modifier preventing reassignment.
- Assignment is only allowed during object creation.

**General Syntax: Combined Optional and Readonly**

```typescript
interface InterfaceName {
  readonly optionalProperty?: Type;
}
```

**Component Breakdown**
- Combines both modifiers: the property is optional and readonly.

**Syntax Rules**

- The `?` modifier can be applied to any property or method.
- The `readonly` modifier can be applied to any property.
- Both modifiers can be combined on the same property.
- Optional properties must be handled with narrowing before use under `strictNullChecks`.
- Readonly properties can only be assigned during object creation.
- Readonly is shallow: nested objects remain mutable.
- `exactOptionalPropertyTypes` distinguishes between "absent" and "undefined" for optional properties.

**Constraints and Limitations**

- Optional properties require narrowing before access under `strictNullChecks`.
- Readonly properties can still be mutated through type assertions or by casting to a mutable type.
- Readonly is erased at runtime and provides no runtime protection.
- Readonly is shallow: nested objects are still mutable unless deep readonly is used.
- `exactOptionalPropertyTypes` may break existing code that assigns `undefined` to optional properties.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Optional and Readonly Properties

```typescript
// Step 1: Declare an interface with optional and readonly properties.
interface UserProfile {
  readonly id: number;
  name: string;
  email?: string;
  readonly createdAt?: Date;
}

// Step 2: Create objects with different combinations.
const user1: UserProfile = { id: 1, name: "Alice" };
const user2: UserProfile = {
  id: 2,
  name: "Bob",
  email: "bob@example.com",
  createdAt: new Date(),
};

// Step 3: Readonly properties can be read.
console.log(`User 1 ID: ${user1.id}`);
console.log(`User 2 email: ${user2.email ?? "(none)"}`);

// Step 4: Readonly properties cannot be reassigned.
// user1.id = 10;  // ❌ Error: Cannot assign to 'id' because it is a read-only property.

// Step 5: Non-readonly properties can be reassigned.
user1.name = "Alice Smith";  // ✅ Allowed
console.log(`Updated name: ${user1.name}`);

// Step 6: Optional properties require narrowing.
function getEmail(user: UserProfile): string {
  return user.email ?? "no email";
}

console.log(getEmail(user1));  // "no email"
console.log(getEmail(user2));  // "bob@example.com"
```

**Expected Output:**
```
User 1 ID: 1
User 2 email: bob@example.com
Updated name: Alice Smith
no email
bob@example.com
```

**Why This Output Occurs:** The `id` and `createdAt` properties are readonly, preventing reassignment. The `email` property is optional, requiring the `??` operator for safe access. The `name` property is mutable and can be reassigned.

### Real-World Cases

**Case 1: Configuration Objects**
Configuration interfaces use optional properties for settings that have defaults and readonly properties for values that should not change at runtime.

**Case 2: API Entities**
API entity interfaces use readonly for IDs and creation timestamps, and optional for fields that may not be present in all responses.

**Case 3: React Props**
React component props use optional modifiers for props with defaults and readonly for props that should not be mutated by the component.

---

## 3. Method Definitions vs Function Property Syntax

### Definitions

**Core Definition**
Interfaces can declare callable members using two syntaxes: method syntax (`methodName(params): ReturnType`) and function property syntax (`methodName: (params) => ReturnType`). Both describe callable members, but they have subtle differences in bivariance and type checking behavior.

**Technical Definition**
Method syntax in interfaces uses the form `name(params): ReturnType`, which is shorthand for a function type with the same signature. Function property syntax uses `name: (params) => ReturnType`, explicitly declaring the property as a function type. The key difference is that method syntax parameters are checked bivariantly (contravariant and covariant), while function property syntax parameters are checked strictly contravariantly under `strictFunctionTypes`. This means method syntax is more permissive with parameter types, which is often desirable for callbacks but can hide type errors. Both syntaxes are structurally compatible with each other in most cases.

**Beginner-Friendly Explanation**
You can write methods in an interface in two ways: as a method (`greet(): string`) or as a function property (`greet: () => string`). They look almost the same and work almost the same. The difference is subtle: methods are checked more loosely (they allow more types in parameters), while function properties are checked more strictly. For most code, either works. Use method syntax when you want the flexibility of bivariant parameter checking (common for callbacks and event handlers). Use function property syntax when you want strict parameter type checking.

### Purposes

- To declare callable members using either concise method syntax or explicit function property syntax.
- To control the variance behavior of function parameters (bivariant for methods, contravariant for properties).
- To document the intended usage of callable members.
- To enable structural compatibility with classes and object literals that use either syntax.
- To provide clarity when the distinction matters for type safety.

### Syntax Rules and Structure

**General Syntax: Method Syntax**

```typescript
interface InterfaceName {
  methodName(param1: Type1, param2: Type2): ReturnType;
}
```

**Component Breakdown**
- `methodName`: The method name.
- `(param1: Type1, param2: Type2)`: The parameter list with types.
- `: ReturnType`: The return type.
- Parameters are checked bivariantly.

**General Syntax: Function Property Syntax**

```typescript
interface InterfaceName {
  functionProperty: (param1: Type1, param2: Type2) => ReturnType;
}
```

**Component Breakdown**
- `functionProperty`: The property name.
- `: (params) => ReturnType`: The function type annotation.
- Parameters are checked strictly contravariantly under `strictFunctionTypes`.

**Syntax Rules**

- Both syntaxes declare callable members.
- Method syntax uses `name(params): ReturnType`.
- Function property syntax uses `name: (params) => ReturnType`.
- Method syntax parameters are bivariant; function property parameters are contravariant under `strictFunctionTypes`.
- Both syntaxes support optional (`?`) modifiers.
- Both syntaxes support generic type parameters (method syntax only).
- Both syntaxes are structurally compatible in most cases.
- Classes implementing interfaces can use either syntax.

**Constraints and Limitations**

- Method syntax cannot be used for overloaded function types in the same way as function properties.
- Function property syntax is more verbose but more explicit.
- The variance difference can cause surprising type errors when mixing syntaxes.
- Method syntax parameters are always bivariant, even under `strictFunctionTypes`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Both Syntaxes in Action

```typescript
// Step 1: Define an interface using both method and function property syntax.
interface Calculator {
  // Method syntax
  add(a: number, b: number): number;

  // Function property syntax
  subtract: (a: number, b: number) => number;
}

// Step 2: Implement the interface with an object literal.
const calc: Calculator = {
  add(a, b) {
    return a + b;
  },
  subtract(a, b) {
    return a - b;
  },
};

// Step 3: Call both methods.
console.log(`Add: ${calc.add(5, 3)}`);         // 8
console.log(`Subtract: ${calc.subtract(5, 3)}`); // 2

// Step 4: Both syntaxes are interchangeable in most cases.
interface Alternative {
  add: (a: number, b: number) => number;
  subtract(a: number, b: number): number;
}

const calc2: Alternative = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
};

console.log(`Add: ${calc2.add(10, 5)}`);        // 15
console.log(`Subtract: ${calc2.subtract(10, 5)}`); // 5
```

**Expected Output:**
```
Add: 8
Subtract: 2
Add: 15
Subtract: 5
```

**Why This Output Occurs:** Both method syntax and function property syntax describe callable members. The object literal implementations are structurally compatible with both syntaxes. The types of `add` and `subtract` are functions in both cases.

#### Example 2: Variance Difference

```typescript
// Step 1: Define an interface with both syntaxes.
interface Handler {
  handleMethod(event: MouseEvent): void;
  handleProperty: (event: MouseEvent) => void;
}

// Step 2: Create a handler that accepts a more general event type.
// Method syntax allows bivariant assignment.
const handler: Handler = {
  handleMethod(event: Event) {  // ✅ Allowed — bivariant
    console.log(`Method: ${event.type}`);
  },
  handleProperty(event: Event) {  // ❌ Error under strictFunctionTypes
    console.log(`Property: ${event.type}`);
  },
};
```

**Expected Output:** The code produces a compile error on `handleProperty` under `strictFunctionTypes`. The `handleMethod` assignment is allowed because method syntax is bivariant.

**Why This Output Occurs:** Method syntax parameters are checked bivariantly, allowing a function accepting a supertype (`Event`) to be assigned to a method expecting a subtype (`MouseEvent`). Function property syntax under `strictFunctionTypes` requires contravariant parameters, so a function accepting a supertype is not assignable to a property expecting a subtype.

### Real-World Cases

**Case 1: Event Handlers**
Event handler interfaces use method syntax to allow flexible event type handling, especially when handlers accept general event types.

**Case 2: Callback Interfaces**
Callback interfaces often use function property syntax to enforce strict parameter type checking, preventing subtle type errors.

**Case 3: Class Contracts**
Interfaces that classes implement often use method syntax because it aligns with class method declaration syntax.

---

## 4. Function and Construct Signatures in Interfaces

### Definitions

**Core Definition**
Interfaces can describe callable objects (functions) via call signatures and newable objects (constructors) via construct signatures. A call signature describes the parameters and return type when the object is called as a function. A construct signature describes the parameters and instance type when the object is used with the `new` keyword.

**Technical Definition**
Call signatures in interfaces use the syntax `(params): ReturnType` without a method name. Construct signatures use `new (params): InstanceType`. Both can appear alongside regular properties in an interface, allowing an object to have both data properties and callable/constructable behavior. Call signatures can be overloaded by declaring multiple signatures. Construct signatures enable typing of factory functions, class constructors, and mixin patterns. These signatures make interfaces capable of describing functions and classes, not just plain objects.

**Beginner-Friendly Explanation**
Most interfaces describe plain objects with properties. But some interfaces describe functions—objects you can call. To do this, you use a "call signature," which looks like a function declaration without a name. For example, `interface Greet { (name: string): string }` describes a function that takes a string and returns a string. Similarly, a "construct signature" describes something you can use with `new`. These are useful for typing functions, classes, and factories. You can even combine a call signature with properties: a function that also has properties.

### Purposes

- To type callable objects (functions) with specific parameter and return types.
- To type constructable objects (classes, factories) with specific constructor signatures.
- To describe hybrid objects that are both callable and have properties.
- To enable overloading by declaring multiple call signatures.
- To type mixins and class factories in advanced patterns.

### Syntax Rules and Structure

**General Syntax: Call Signature**

```typescript
interface CallableInterface {
  (param1: Type1, param2: Type2): ReturnType;
}
```

**Component Breakdown**
- `(param1: Type1, param2: Type2): ReturnType`: The call signature.
- No name is provided; the interface itself describes the function.

**General Syntax: Construct Signature**

```typescript
interface ConstructableInterface {
  new (param1: Type1, param2: Type2): InstanceType;
}
```

**Component Breakdown**
- `new (params): InstanceType`: The construct signature.
- `new`: The keyword indicating constructability.
- `InstanceType`: The type of the constructed instance.

**General Syntax: Combined Properties and Signatures**

```typescript
interface Hybrid {
  (input: string): number;       // Call signature
  new (input: string): object;   // Construct signature
  version: string;                // Property
}
```

**Component Breakdown**
- The object is callable, constructable, and has a property.

**Syntax Rules**

- Call signatures have no name and use `(params): ReturnType` syntax.
- Construct signatures use `new (params): InstanceType` syntax.
- Multiple call signatures create overloads.
- Call and construct signatures can coexist with properties.
- Construct signatures cannot have type parameters directly (use a generic interface instead).
- Call signatures can be generic at the interface level.

**Constraints and Limitations**

- Call and construct signatures cannot have names (use method syntax if a name is needed).
- Construct signatures do not describe static members of classes.
- Overloaded call signatures must be ordered from most specific to least specific.
- Call and construct signatures are erased at runtime.
- Hybrid interfaces (with both signatures and properties) are uncommon and may confuse readers.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Call Signature

```typescript
// Step 1: Define an interface with a call signature.
interface StringFormatter {
  (input: string): string;
}

// Step 2: Implement the interface with a function.
const uppercase: StringFormatter = (input) => input.toUpperCase();
const exclaim: StringFormatter = (input) => `${input}!`;

// Step 3: Call the functions.
console.log(uppercase("hello"));  // "HELLO"
console.log(exclaim("hello"));    // "hello!"

// Step 4: Combine with properties.
interface DescribableFormatter {
  (input: string): string;
  description: string;
}

const trim: DescribableFormatter = (input) => input.trim();
trim.description = "Trims whitespace";

console.log(trim("  hello  "));    // "hello"
console.log(trim.description);      // "Trims whitespace"
```

**Expected Output:**
```
HELLO
hello!
hello
Trims whitespace
```

**Why This Output Occurs:** The `StringFormatter` interface describes a callable object. The `uppercase` and `exclaim` functions satisfy the call signature. The `DescribableFormatter` combines a call signature with a property, allowing the function to have a `description`.

#### Example 2: Construct Signature

```typescript
// Step 1: Define an interface with a construct signature.
interface UserConstructor {
  new (name: string, age: number): { name: string; age: number; greet(): string };
}

// Step 2: Create a class that matches the construct signature.
class User {
  constructor(public name: string, public age: number) {}
  greet(): string {
    return `Hi, I'm ${this.name}`;
  }
}

// Step 3: Use the constructor interface.
function createUser(ctor: UserConstructor, name: string, age: number) {
  return new ctor(name, age);
}

const user = createUser(User, "Alice", 30);
console.log(user.greet());  // "Hi, I'm Alice"

// Step 4: Factory functions can also satisfy construct signatures.
function makeUser(name: string, age: number) {
  return {
    name,
    age,
    greet() {
      return `Hello, ${name}`;
    },
  };
}

// Note: A factory function is not constructable with `new`, so it does NOT
// satisfy the UserConstructor interface directly. Use a class or a
// constructable function.
```

**Expected Output:**
```
Hi, I'm Alice
```

**Why This Output Occurs:** The `UserConstructor` interface describes a constructor that takes `name` and `age` and returns an instance with those properties and a `greet` method. The `User` class satisfies this interface. The `createUser` function accepts any constructor matching the interface, enabling dependency injection and factory patterns.

### Real-World Cases

**Case 1: Factory Patterns**
Factory functions and dependency injection containers use construct signatures to type class constructors, enabling flexible object creation.

**Case 2: Middleware and Plugins**
Plugin systems use call signatures to type plugin functions, ensuring they match the expected signature.

**Case 3: Function Libraries**
Libraries like Lodash use call signatures to type their utility functions, enabling type-safe functional programming.

---

## 5. Interface Extension (extends with Single/Multiple Interfaces)

### Definitions

**Core Definition**
Interface extension allows an interface to inherit members from one or more parent interfaces using the `extends` clause. The child interface includes all members of its parents plus any additional members it declares. Multiple inheritance is supported by listing multiple parent interfaces separated by commas.

**Technical Definition**
The `extends` clause in an interface declaration creates a subtype relationship: the child interface is assignable to each parent interface. The child interface inherits all members (properties, methods, call signatures, construct signatures, and index signatures) from its parents. If two parents declare the same property with incompatible types, a compile error occurs. If they declare the same property with compatible types, the child inherits the more specific type (or the intersection). Interfaces can extend classes (inheriting the class's instance members) but cannot extend type aliases representing unions. Interfaces can also be extended by type aliases via intersections.

**Beginner-Friendly Explanation**
Interface extension lets one interface build on another. If you have a `Person` interface with `name` and `age`, you can create an `Employee` interface that extends `Person` and adds an `employeeId`. The `Employee` interface automatically has `name`, `age`, and `employeeId`. You can extend multiple interfaces at once: `interface Manager extends Employee, Approver` gives `Manager` all members of both. This is like inheritance in classes but for interfaces. It helps avoid repetition and creates clear type hierarchies.

### Purposes

- To create specialized interfaces that build on more general ones.
- To reuse common properties across related interfaces.
- To model type hierarchies (e.g., `Animal` → `Dog` → `ServiceDog`).
- To enable multiple inheritance of interface contracts.
- To facilitate polymorphism and substitutability in function parameters.

### Syntax Rules and Structure

**General Syntax: Single Extension**

```typescript
interface ChildInterface extends ParentInterface {
  additionalProperty: Type;
}
```

**Component Breakdown**
- `extends ParentInterface`: The parent interface to inherit from.
- The child inherits all parent members and adds its own.

**General Syntax: Multiple Extension**

```typescript
interface ChildInterface extends Parent1, Parent2, Parent3 {
  additionalProperty: Type;
}
```

**Component Breakdown**
- Multiple parents are listed after `extends`, separated by commas.
- The child inherits all members from all parents.

**General Syntax: Extending a Class**

```typescript
interface InterfaceName extends ClassName {
  additionalProperty: Type;
}
```

**Component Breakdown**
- Interfaces can extend classes, inheriting the class's instance members (not static members or constructors).

**Syntax Rules**

- The `extends` clause follows the interface name (and generic parameters).
- Multiple parents are separated by commas.
- The child interface inherits all members from parents.
- If parents have conflicting property types, a compile error occurs.
- Interfaces can extend classes (inheriting instance members only).
- Interfaces cannot extend type aliases representing unions.
- The child interface can override parent members with compatible types (narrowing).
- Generic parent interfaces must have their type arguments specified in the extends clause.

**Constraints and Limitations**

- Conflicting member types across parents cause compile errors.
- Interfaces cannot extend union types or primitive types.
- Extending a class with private or protected members makes the interface nominal for those members.
- The child interface cannot remove members from parents (only add or narrow).
- Deep extension hierarchies can become difficult to understand.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Single and Multiple Extension

```typescript
// Step 1: Define base interfaces.
interface Person {
  name: string;
  age: number;
}

interface Contactable {
  email: string;
  phone?: string;
}

// Step 2: Extend a single interface.
interface Employee extends Person {
  employeeId: number;
  department: string;
}

// Step 3: Extend multiple interfaces.
interface Manager extends Person, Contactable {
  reports: Employee[];
  approve(requestId: number): boolean;
}

// Step 4: Create objects that satisfy the extended interfaces.
const alice: Employee = {
  name: "Alice",
  age: 30,
  employeeId: 1001,
  department: "Engineering",
};

const bob: Manager = {
  name: "Bob",
  age: 40,
  email: "bob@example.com",
  reports: [alice],
  approve(requestId) {
    return requestId > 0;
  },
};

// Step 5: Substitutability — a Manager is a Person and Contactable.
function describePerson(person: Person): void {
  console.log(`${person.name}, ${person.age}`);
}

describePerson(alice);  // ✅ Works — Employee is a Person
describePerson(bob);    // ✅ Works — Manager is a Person

// Step 6: Access all inherited members.
console.log(bob.email);           // "bob@example.com"
console.log(bob.reports.length);  // 1
console.log(bob.approve(42));     // true
```

**Expected Output:**
```
Alice, 30
Bob, 40
bob@example.com
1
true
```

**Why This Output Occurs:** `Employee` extends `Person`, inheriting `name` and `age`. `Manager` extends both `Person` and `Contactable`, inheriting `name`, `age`, `email`, and `phone?`. The `describePerson` function accepts any `Person`, and both `alice` (Employee) and `bob` (Manager) are assignable because they have all `Person` members.

#### Example 2: Overriding and Narrowing

```typescript
// Step 1: Define a base interface.
interface Animal {
  name: string;
  makeSound(): string;
}

// Step 2: Extend and narrow the return type.
interface Dog extends Animal {
  breed: string;
  makeSound(): "Woof";  // Narrowed return type
}

// Step 3: Extend further.
interface ServiceDog extends Dog {
  task: string;
  makeSound(): "Woof";  // Still compatible with Dog
}

// Step 4: Create objects.
const dog: Dog = {
  name: "Rex",
  breed: "German Shepherd",
  makeSound: () => "Woof",
};

const serviceDog: ServiceDog = {
  name: "Buddy",
  breed: "Labrador",
  task: "Guide",
  makeSound: () => "Woof",
};

console.log(dog.makeSound());        // "Woof"
console.log(serviceDog.task);        // "Guide"

// Step 5: Both are assignable to Animal.
const animals: Animal[] = [dog, serviceDog];
animals.forEach((a) => console.log(`${a.name}: ${a.makeSound()}`));
```

**Expected Output:**
```
Woof
Guide
Rex: Woof
Buddy: Woof
```

**Why This Output Occurs:** `Dog` extends `Animal` and narrows the return type of `makeSound` from `string` to `"Woof"` (a subtype of `string`). `ServiceDog` extends `Dog` and remains compatible. All types are assignable to `Animal` because they satisfy the base contract.

### Real-World Cases

**Case 1: Domain Hierarchies**
E-commerce systems use interface extension to model domain hierarchies: `Product` → `PhysicalProduct` → `ShippableProduct`.

**Case 2: React Component Props**
React components use interface extension to compose props: `BaseButtonProps` extended by `PrimaryButtonProps` and `IconButtonProps`.

**Case 3: API Response Types**
API response types extend base response interfaces: `ApiResponse<T>` → `PaginatedResponse<T>` → `FilteredResponse<T>`.

---

## 6. Declaration Merging (Interface Merging Behavior)

### Definitions

**Core Definition**
Declaration merging is a TypeScript feature where multiple declarations with the same name in the same scope are combined into a single declaration. Interfaces are the primary beneficiary: declaring the same interface name multiple times merges all members into one interface. This enables module augmentation, extending third-party types, and splitting interface definitions across files.

**Technical Definition**
Interface declaration merging combines the members of all declarations with the same name. Members are merged with the following rules: non-function members must be unique across declarations (duplicate identical types are allowed; conflicting types are errors), while function members with the same name are overloaded (later declarations take precedence in overload resolution). Interfaces can merge with classes and namespaces (but not with type aliases or variables). Merging across module boundaries requires the `declare global` or module augmentation pattern. Declaration merging is a key mechanism for extending library types without forking.

**Beginner-Friendly Explanation**
Declaration merging means you can declare the same interface multiple times, and TypeScript combines them. If you write `interface User { name: string }` in one file and `interface User { email: string }` in another (same scope), TypeScript treats them as a single `User` interface with both `name` and `email`. This is useful for extending library types without modifying them—you can add properties to a library's interface by declaring it again. It's also useful for splitting large interfaces across files for organization. But be careful: conflicting property types cause errors.

### Purposes

- To split large interface definitions across multiple files for organization.
- To extend third-party library types without modifying their source.
- To augment global types (e.g., adding properties to `Window` or `Array`).
- To enable plugin systems where plugins contribute additional interface members.
- To combine interface declarations from multiple sources into one cohesive type.

### Syntax Rules and Structure

**General Syntax: Interface Merging**

```typescript
// File 1
interface User {
  name: string;
}

// File 2 (same scope)
interface User {
  email: string;
}

// Result: interface User { name: string; email: string; }
```

**Component Breakdown**
- Multiple declarations with the same name are merged.
- Non-function members must have unique names (or identical types).
- Function members with the same name are overloaded.

**General Syntax: Module Augmentation**

```typescript
// augment.ts
import "some-library";

declare module "some-library" {
  interface LibraryInterface {
    newProperty: string;
  }
}
```

**Component Breakdown**
- `declare module "some-library"`: Augments the module's types.
- `interface LibraryInterface`: Merges with the library's interface.

**General Syntax: Global Augmentation**

```typescript
declare global {
  interface Window {
    myCustomProperty: string;
  }
}

export {};
```

**Component Breakdown**
- `declare global`: Declares global augmentations.
- `interface Window`: Merges with the global `Window` interface.

**Syntax Rules**

- Multiple interface declarations with the same name in the same scope are merged.
- Non-function members must not conflict (identical types are allowed).
- Function members with the same name become overloads.
- Later declarations' overloads take precedence.
- Interfaces can merge with classes (adding instance members).
- Interfaces can merge with namespaces.
- Interfaces cannot merge with type aliases or variables.
- Module augmentation requires `declare module` inside a module file.
- Global augmentation requires `declare global` and an `export {}` to make the file a module.

**Constraints and Limitations**

- Non-function members with conflicting types cause compile errors.
- Merging cannot be prevented (all declarations with the same name merge).
- Module augmentation cannot add new top-level exports, only modify existing ones.
- Global augmentation is discouraged for application code (only for libraries).
- Declaration merging can make types harder to understand (members are spread across declarations).
- Type aliases cannot participate in declaration merging.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Interface Merging

```typescript
// Step 1: First declaration of the User interface.
interface User {
  id: number;
  name: string;
}

// Step 2: Second declaration — merges with the first.
interface User {
  email: string;
  age?: number;
}

// Step 3: The merged interface has all members.
const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  age: 30,
};

console.log(user.name);   // "Alice"
console.log(user.email);  // "alice@example.com"

// Step 4: All properties are required (except optional ones).
// const invalid: User = { id: 1, name: "Bob" };
// ❌ Error: Property 'email' is missing.

// Step 5: Function members merge as overloads.
interface Calculator {
  add(a: number, b: number): number;
}

interface Calculator {
  add(a: string, b: string): string;
}

const calc: Calculator = {
  add(a: any, b: any) {
    return a + b;
  },
};

console.log(calc.add(1, 2));       // 3
console.log(calc.add("a", "b"));   // "ab"
```

**Expected Output:**
```
Alice
alice@example.com
3
ab
```

**Why This Output Occurs:** The two `User` declarations merge into one interface with `id`, `name`, `email`, and `age?`. The two `Calculator` declarations merge, and the `add` method becomes an overloaded function accepting both number and string pairs.

#### Example 2: Module Augmentation

```typescript
// Step 1: Assume a library defines an interface.
// library.d.ts:
// export interface Config {
//   apiUrl: string;
// }

// Step 2: Augment the library's interface in your code.
// augment.ts:
import { Config } from "some-library";

declare module "some-library" {
  interface Config {
    retries?: number;
    timeout?: number;
  }
}

// Step 3: Now Config has the additional properties.
const config: Config = {
  apiUrl: "https://api.example.com",
  retries: 3,
  timeout: 5000,
};

console.log(config.apiUrl);   // "https://api.example.com"
console.log(config.retries);  // 3

// Step 4: Global augmentation (e.g., adding to Window).
// global.d.ts:
declare global {
  interface Window {
    __APP_VERSION__: string;
  }
}
export {};

// Step 5: Usage in application code.
window.__APP_VERSION__ = "1.0.0";
console.log(window.__APP_VERSION__);  // "1.0.0"
```

**Expected Output:**
```
https://api.example.com
3
1.0.0
```

**Why This Output Occurs:** Module augmentation adds `retries?` and `timeout?` to the library's `Config` interface without modifying the library. Global augmentation adds `__APP_VERSION__` to the global `Window` interface. Both augmentations merge with the original declarations.

### Real-World Cases

**Case 1: Extending Library Types**
When using libraries like Express or Passport, developers augment the `Request` interface to add custom properties like `req.user` without modifying the library.

**Case 2: Plugin Systems**
Plugin systems use interface merging to allow plugins to contribute additional properties to a shared context interface, enabling extensibility without modifying core types.

**Case 3: Splitting Large Interfaces**
Large interfaces (e.g., a 50-property configuration) can be split across multiple files by declaring the interface in each file, keeping each file focused and manageable.

---

## References

- TypeScript Handbook: Interfaces — https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook: Declaration Merging — https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript Handbook: More on Functions — https://www.typescriptlang.org/docs/handbook/2/functions.html
- TypeScript Handbook: Classes — https://www.typescriptlang.org/docs/handbook/2/classes.html
- TypeScript 4.4 Release Notes (`exactOptionalPropertyTypes`) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html
- TypeScript Wiki: FAQ — Interface vs Type Alias — https://github.com/microsoft/TypeScript/wiki/FAQ
- Effective TypeScript: Item 13 — Know the Differences Between type and interface
- TypeScript Playground: Interfaces — https://www.typescriptlang.org/play/typescript/interfaces.ts.html
- Total TypeScript: Interfaces vs Types — https://www.totaltypescript.com/type-vs-interface
- TypeScript ESLint: consistent-type-definitions — https://typescript-eslint.io/rules/consistent-type-definitions/
- MDN: Object.prototype — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object