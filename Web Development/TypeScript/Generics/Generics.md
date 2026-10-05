# TypeScript Generic Fundamentals: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Generics in TypeScript are a language feature that allows you to write reusable, type-safe code by parameterizing types. A generic type is a "template" that works with a variety of types while preserving the specific type information of the values it operates on. Instead of writing separate functions or classes for each type, you write one generic component that adapts to the type it's used with.

**Technical Definition**
Generics in TypeScript are implemented through type parameters—special kinds of variables that work on types rather than values. A generic declaration introduces one or more type parameters in angle brackets (e.g., `<T>`), which serve as placeholders for actual types provided at the use site. TypeScript employs a constraint-based type inference system: when a generic function is called, the compiler infers type arguments from the values passed in, unless explicitly provided. Generic types are "templates" from which multiple concrete types can be instantiated. The type parameters of a generic declaration are in scope throughout that declaration and can be constrained using the `extends` keyword. Generic parameter defaults are supported, and type parameters are erased at runtime.

**Beginner-Friendly Explanation**
Generics are TypeScript's way of writing code that works with many types while keeping full type safety. Think of them like function parameters, but for types. If you write a function `identity<T>(value: T): T`, the `T` is a placeholder for whatever type you pass in. When you call `identity(42)`, TypeScript knows `T` is `number`, and you get a `number` back. When you call `identity("hello")`, `T` becomes `string`. Without generics, you'd either write separate functions for each type or use `any` and lose all type information. Generics let you write once and use with any type, while TypeScript still tracks the specific type at each call site.

### Key Characteristics

- **Type parameterization**: Types are passed as parameters, making components reusable across many types.
- **Type preservation**: Unlike `any`, generics preserve the specific type information at each use site.
- **Inference**: TypeScript infers type arguments from values when not explicitly provided.
- **Constraints**: Type parameters can be constrained with `extends` to require certain shapes.
- **Defaults**: Generic parameters can have default types.
- **Scope**: Type parameters are scoped to the declaration they appear in.
- **Runtime erasure**: Type parameters are erased at runtime; they exist only at compile time.
- **Works across constructs**: Generics apply to functions, interfaces, classes, and type aliases.

### Prerequisites

- Basic knowledge of TypeScript types (primitives, objects, unions)
- Familiarity with functions, interfaces, classes, and type aliases
- Understanding of type annotations and type inference
- Familiarity with `tsconfig.json` compiler options

### Related Programming Areas

- **Type Theory**: Parametric polymorphism, bounded quantification
- **Type-Level Programming**: Generic constraints, conditional types, mapped types
- **Reusable Components**: Library design, utility types, container types
- **Functional Programming**: Generic higher-order functions
- **Object-Oriented Programming**: Generic classes and interfaces

### Core Concepts / Features

1. Generic Type Parameters (Type Variables as Placeholders)
2. Generic Functions and Method Signatures
3. Generic Interfaces for Flexible Object Shapes
4. Generic Classes (Instance Properties, Methods, and Static Limitations)
5. Generic Type Aliases and Generic Parameter Scope
6. Generic Arrow Function Syntax Constraints and the `<T,>` / `T extends unknown` JSX Parsing Workaround


## 1. Generic Type Parameters (Type Variables as Placeholders)

### Definitions

**Core Definition**
Generic type parameters (also called type variables) are identifiers declared in angle brackets (`<T>`) that serve as placeholders for actual types. They allow a single declaration to work with multiple types while preserving type relationships between inputs and outputs.

**Technical Definition**
A type parameter is a special kind of variable that operates on types rather than values. When a generic declaration is used, the type parameter is replaced with a concrete type argument (either explicitly provided or inferred). Type parameters can be constrained with `extends` to require a minimum shape, and can have default types. The scope of a type parameter is the entire declaration in which it appears. TypeScript uses type parameter inference to determine the type argument from the values passed to a generic function or the type arguments provided to a generic type. Type parameters are erased at runtime and exist only during compilation.

**Beginner-Friendly Explanation**
A type parameter is like a placeholder that says "some type goes here." When you write `function identity<T>(value: T): T`, the `T` is a placeholder. You can use `T` anywhere in the function—in parameter types, return types, and local variable types. When you call `identity(42)`, TypeScript replaces `T` with `number`. The type parameter acts like a function parameter, but for types instead of values. This is what makes generics so powerful: you write code once, and it works with any type.

### Purposes

- To create reusable components that work with multiple types.
- To preserve type relationships between inputs and outputs.
- To avoid code duplication across similar type-specific implementations.
- To provide type safety without resorting to `any`.
- To enable type-level abstraction and parameterization.

### Syntax Rules and Structure

**General Syntax: Single Type Parameter**

```typescript
function functionName<T>(param: T): T {
  return param;
}
```

**Component Breakdown**
- `<T>`: The type parameter declaration.
- `T`: The type parameter identifier (conventionally `T`, `U`, `V`, or descriptive names).
- `param: T`: A parameter using the type parameter.
- `): T`: The return type using the type parameter.

**General Syntax: Multiple Type Parameters**

```typescript
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}
```

**Component Breakdown**
- `<T, U>`: Multiple type parameters separated by commas.
- Each parameter can be used independently.

**General Syntax: Constrained Type Parameter**

```typescript
function logLength<T extends { length: number }>(value: T): T {
  console.log(value.length);
  return value;
}
```

**Component Breakdown**
- `<T extends { length: number }>`: The constraint requires `T` to have a `length` property.
- The constraint ensures `value.length` is valid.

**General Syntax: Default Type Parameter**

```typescript
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}
```

**Component Breakdown**
- `<T = string>`: The default type is `string` when `T` cannot be inferred.

**Syntax Rules**

- Type parameters are declared in angle brackets after the function, class, or interface name.
- Type parameter names are conventionally single uppercase letters (`T`, `U`, `V`) or descriptive (`TValue`, `TKey`).
- Multiple type parameters are separated by commas.
- Type parameters can be constrained with `extends`.
- Type parameters can have default types with `=`.
- Type parameters are in scope throughout the declaration.
- Type parameters are erased at runtime.

**Constraints and Limitations**

- Type parameters cannot be used as values (e.g., `new T()` is not allowed).
- Type parameters cannot appear in static members of generic classes.
- Type parameters are erased at runtime; no runtime type information is available.
- Unconstrained type parameters default to `unknown` in strict mode.
- Very complex generic signatures can be difficult to read.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Generic Function

```typescript
// Step 1: Define a generic identity function.
function identity<T>(value: T): T {
  return value;
}

// Step 2: Call with explicit type argument.
const num1 = identity<number>(42);
console.log(num1);  // 42

// Step 3: Call with type inference.
const str1 = identity("hello");
console.log(str1);  // "hello"

// Step 4: TypeScript preserves the type.
const num2: number = identity(100);      // ✅
const str2: string = identity("world");  // ✅
// const wrong: string = identity(42);   // ❌ Error: number not assignable to string.

// Step 5: Without generics, we'd lose type information.
function identityAny(value: any): any {
  return value;
}
const result = identityAny("hello");  // result: any — no type safety!
```

**Expected Output:**
```
42
hello
```

**Why This Output Occurs:** The generic `identity<T>` function captures the type of its argument and returns the same type. When called with `42`, `T` is inferred as `number`. When called with `"hello"`, `T` is inferred as `string`. The `any` version loses all type information.

#### Example 2: Constrained and Default Type Parameters

```typescript
// Step 1: Constrained type parameter.
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(value: T): T {
  console.log(`Length: ${value.length}`);
  return value;
}

logLength("hello");        // "Length: 5"
logLength([1, 2, 3]);      // "Length: 3"
logLength({ length: 10 }); // "Length: 10"
// logLength(42);          // ❌ Error: number does not have 'length'.

// Step 2: Default type parameter.
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

const strings = createArray(3, "hello");  // string[]
const numbers = createArray(3, 42);       // number[]
const defaults = createArray(3, "hi");    // string[] (uses default)

console.log(strings);   // ["hello", "hello", "hello"]
console.log(numbers);   // [42, 42, 42]
```

**Expected Output:**
```
Length: 5
Length: 3
Length: 10
[ 'hello', 'hello', 'hello' ]
[ 42, 42, 42 ]
```

**Why This Output Occurs:** The `extends HasLength` constraint requires the argument to have a `length` property. The default type parameter `T = string` means that if `T` cannot be inferred, it defaults to `string`. The generic function preserves the specific type of the value.

### Real-World Cases

**Case 1: Utility Functions**
Functions like `identity`, `clone`, and `first` use generics to work with any type while preserving type information.

**Case 2: Container Types**
Arrays, Maps, Sets, and custom containers use generics to specify their element types.

**Case 3: API Response Wrappers**
Generic response types like `ApiResponse<T>` wrap data of any type while preserving the payload type.

**Case 4: Type-Safe Event Emitters**
Event emitters use generics to type event names and payloads, ensuring that handlers receive the correct event data.


## 2. Generic Functions and Method Signatures

### Definitions

**Core Definition**
A generic function is a function that declares one or more type parameters on its call signature. The type parameters are specified (or inferred) when the function is called, allowing the function to work with different types while maintaining type relationships between parameters and return values.

**Technical Definition**
A generic function declares its type parameters on the call signature, not on the function itself. The type parameters are in scope throughout the function body and can be used in parameter types, return types, and local variable types. TypeScript infers type arguments from the values passed at the call site unless explicitly provided. Generic methods are functions declared within classes, interfaces, or object literals that have their own type parameters, independent of any type parameters on the containing type.

**Beginner-Friendly Explanation**
A generic function is a function that can work with any type. You declare the type parameters in angle brackets after the function name: `function identity<T>(value: T): T`. When you call the function, TypeScript figures out what `T` should be based on the arguments you pass. Generic methods work the same way but are defined inside a class or object. The key benefit is that the function remembers the specific type you used, so you get type safety without writing separate functions for each type.

### Purposes

- To write reusable functions that work with multiple types.
- To preserve type relationships between parameters and return values.
- To enable type-safe higher-order functions.
- To avoid code duplication and `any` usage.
- To provide type-safe utility functions and algorithms.

### Syntax Rules and Structure

**General Syntax: Generic Function Declaration**

```typescript
function functionName<T>(param: T): T {
  return param;
}
```

**Component Breakdown**
- `<T>`: Type parameter declared on the function.
- `param: T`: Parameter using the type parameter.
- `): T`: Return type using the type parameter.

**General Syntax: Generic Function Expression**

```typescript
const functionName = function <T>(param: T): T {
  return param;
};
```

**Component Breakdown**
- Type parameters are declared after the `function` keyword.

**General Syntax: Generic Method in Interface**

```typescript
interface Container {
  get<T>(index: number): T;
  set<T>(index: number, value: T): void;
}
```

**Component Breakdown**
- Methods can have their own type parameters independent of the interface.

**General Syntax: Generic Method in Class**

```typescript
class Collection {
  private items: any[] = [];

  add<T>(item: T): void {
    this.items.push(item);
  }

  get<T>(index: number): T {
    return this.items[index];
  }
}
```

**Component Breakdown**
- Generic methods have their own type parameters.

**Syntax Rules**

- Type parameters are declared in angle brackets after the function name.
- Type parameters are in scope throughout the function body.
- Type arguments are inferred from the call site unless explicitly provided.
- Multiple type parameters are separated by commas.
- Generic methods can be declared in classes, interfaces, and object literals.
- Methods can have their own type parameters independent of the containing type.
- Type parameters can be constrained with `extends`.

**Constraints and Limitations**

- Generic methods cannot access static members of generic classes.
- Type parameters are erased at runtime.
- Overloaded generic functions require careful declaration.
- Type inference may not always produce the expected type; explicit annotations can help.
- Generic methods in classes cannot reference the class's type parameters if they are static.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Generic Function

```typescript
// Step 1: Define a generic function.
function firstElement<T>(array: T[]): T | undefined {
  return array[0];
}

// Step 2: Call with different types.
const firstNum = firstElement([1, 2, 3]);        // number | undefined
const firstStr = firstElement(["a", "b"]);       // string | undefined
const firstObj = firstElement([{ name: "Alice" }]); // { name: string } | undefined

console.log(firstNum);  // 1
console.log(firstStr);  // "a"

// Step 3: Explicit type argument.
const explicit = firstElement<number>([10, 20, 30]);
console.log(explicit);  // 10

// Step 4: Type safety is preserved.
// const wrong: string = firstElement([1, 2, 3]);
// ❌ Error: number is not assignable to string.
```

**Expected Output:**
```
1
a
10
```

**Why This Output Occurs:** The generic `firstElement<T>` function infers `T` from the array argument. When called with `[1, 2, 3]`, `T` is `number`. When called with `["a", "b"]`, `T` is `string`. The return type is `T | undefined` because the array may be empty.

#### Example 2: Generic Method in a Class

```typescript
// Step 1: Define a class with generic methods.
class Stack {
  private items: unknown[] = [];

  push<T>(item: T): void {
    this.items.push(item);
  }

  pop<T>(): T | undefined {
    return this.items.pop() as T | undefined;
  }

  // Generic method with constraint.
  find<T extends { id: number }>(id: number): T | undefined {
    return this.items.find(
      (item): item is T => (item as T).id === id
    ) as T | undefined;
  }
}

// Step 2: Use the generic methods.
const stack = new Stack();
stack.push("hello");
stack.push(42);
stack.push({ id: 1, name: "Alice" });

const str = stack.pop<string>();
console.log(str);  // { id: 1, name: "Alice" } — actually the last pushed

// Step 3: Constrained generic method.
stack.push({ id: 2, name: "Bob" });
const found = stack.find<{ id: number; name: string }>(2);
console.log(found);  // { id: 2, name: 'Bob' }
```

**Expected Output:**
```
{ id: 1, name: 'Alice' }
{ id: 2, name: 'Bob' }
```

**Why This Output Occurs:** The `Stack` class has generic methods `push<T>`, `pop<T>`, and `find<T extends { id: number }>`. Each method has its own type parameter, independent of the class. The `find` method uses a constraint to require an `id` property. TypeScript infers the return types based on the type arguments.

### Real-World Cases

**Case 1: Array Utility Functions**
Functions like `map`, `filter`, `reduce`, and `find` are generic, enabling type-safe array operations.

**Case 2: Promise Handling**
`Promise<T>` and `async` functions are generic, preserving the resolved value type.

**Case 3: Event Handlers**
Event handler functions use generics to type event payloads while preserving event-specific data.

**Case 4: API Client Methods**
API client methods use generics to type request and response payloads, ensuring type safety across network calls.


## 3. Generic Interfaces for Flexible Object Shapes

### Definitions

**Core Definition**
A generic interface is an interface declaration that declares one or more type parameters. The type parameters act as placeholders for types that are supplied when the interface is used, allowing the interface to describe object shapes that work with different types.

**Technical Definition**
A generic interface declares type parameters in angle brackets after the interface name. The type parameters are in scope throughout the interface body and can be used in property types, method signatures, and index signatures. Generic interfaces are "templates" from which multiple concrete interfaces can be instantiated by providing type arguments. Type arguments can be inferred in some contexts (e.g., when a generic interface is used as a function parameter type) but are often explicitly provided. Generic interfaces are commonly used for container types, repository patterns, and API response wrappers.

**Beginner-Friendly Explanation**
A generic interface is an interface that takes type parameters. For example, `interface Box<T> { value: T }` describes a box that holds a value of any type. When you write `Box<number>`, you get an interface with `value: number`. When you write `Box<string>`, you get `value: string`. Generic interfaces let you define reusable object shapes that work with different types. They're especially useful for containers, wrappers, and patterns where the shape is the same but the data type varies.

### Purposes

- To define reusable object shapes that work with multiple types.
- To create type-safe container and wrapper interfaces.
- To enable repository and data-access patterns with type-safe entities.
- To provide flexible API response and request types.
- To support generic data structures and collections.

### Syntax Rules and Structure

**General Syntax: Generic Interface**

```typescript
interface InterfaceName<T> {
  property: T;
  method(param: T): T;
}
```

**Component Breakdown**
- `<T>`: Type parameter declared after the interface name.
- `property: T`: Property using the type parameter.
- `method(param: T): T`: Method using the type parameter.

**General Syntax: Multiple Type Parameters**

```typescript
interface Pair<T, U> {
  first: T;
  second: U;
}
```

**Component Breakdown**
- `<T, U>`: Multiple type parameters.

**General Syntax: Constrained Generic Interface**

```typescript
interface Repository<T extends { id: number }> {
  findById(id: number): T | undefined;
  save(entity: T): void;
}
```

**Component Breakdown**
- `<T extends { id: number }>`: Constraint requiring an `id` property.

**General Syntax: Default Type Parameter**

```typescript
interface ApiResponse<T = unknown> {
  data: T;
  status: number;
}
```

**Component Breakdown**
- `<T = unknown>`: Default type is `unknown`.

**Syntax Rules**

- Type parameters are declared in angle brackets after the interface name.
- Type parameters are in scope throughout the interface body.
- Generic interfaces are used with type arguments: `InterfaceName<Type>`.
- Type arguments can be explicit or inferred in some contexts.
- Type parameters can be constrained with `extends`.
- Type parameters can have default types with `=`.
- Generic interfaces can extend other generic interfaces.

**Constraints and Limitations**

- Type parameters are erased at runtime.
- Generic interfaces cannot be instantiated directly (they describe shapes).
- Type inference for generic interfaces is limited compared to generic functions.
- Complex generic interfaces can be difficult to read and maintain.
- Generic interfaces cannot have static members.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Generic Container Interface

```typescript
// Step 1: Define a generic container interface.
interface Container<T> {
  value: T;
  getValue(): T;
  map<U>(fn: (value: T) => U): Container<U>;
}

// Step 2: Implement the interface.
function createContainer<T>(value: T): Container<T> {
  return {
    value,
    getValue() { return value; },
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

// Step 4: Type safety is preserved.
const num: number = numberContainer.getValue();      // ✅
const str: string = stringContainer.getValue();      // ✅
// const wrong: string = numberContainer.getValue(); // ❌ Error.
```

**Expected Output:**
```
42
42
[ 42, 84 ]
```

**Why This Output Occurs:** The `Container<T>` interface is generic over its value type. The `map<U>` method transforms the container's value from `T` to `U`, preserving type safety. Each container instance knows its value type.

#### Example 2: Generic Repository Interface

```typescript
// Step 1: Define a base entity interface.
interface Entity {
  id: number;
}

// Step 2: Define a generic repository interface with constraint.
interface Repository<T extends Entity> {
  findById(id: number): T | undefined;
  findAll(): T[];
  save(entity: T): void;
  delete(id: number): void;
}

// Step 3: Implement for a specific entity.
interface User extends Entity {
  name: string;
  email: string;
}

class UserRepository implements Repository<User> {
  private users: Map<number, User> = new Map();

  findById(id: number): User | undefined {
    return this.users.get(id);
  }

  findAll(): User[] {
    return Array.from(this.users.values());
  }

  save(user: User): void {
    this.users.set(user.id, user);
  }

  delete(id: number): void {
    this.users.delete(id);
  }
}

// Step 4: Use the repository.
const repo: Repository<User> = new UserRepository();
repo.save({ id: 1, name: "Alice", email: "alice@example.com" });
repo.save({ id: 2, name: "Bob", email: "bob@example.com" });

console.log(repo.findById(1));  // { id: 1, name: 'Alice', email: 'alice@example.com' }
console.log(repo.findAll().length);  // 2
```

**Expected Output:**
```
{ id: 1, name: 'Alice', email: 'alice@example.com' }
2
```

**Why This Output Occurs:** The `Repository<T extends Entity>` interface is generic over the entity type, constrained to have an `id` property. `UserRepository` implements `Repository<User>`, providing type-safe CRUD operations for `User` entities.

### Real-World Cases

**Case 1: API Response Wrappers**
`ApiResponse<T>` interfaces wrap API responses with metadata while preserving the payload type.

**Case 2: Repository Pattern**
Generic repositories (`Repository<T>`) provide CRUD operations for any entity type, reducing boilerplate.

**Case 3: State Management**
State containers (`Store<T>`) hold application state of any type while preserving type safety.

**Case 4: Form Libraries**
Form libraries use generic interfaces to type form values and validation results.


## 4. Generic Classes (Instance Properties, Methods, and Static Limitations)

### Definitions

**Core Definition**
A generic class is a class declaration that declares one or more type parameters. The type parameters are in scope throughout the class body and can be used in instance properties, method signatures, and constructor parameters. Static members, however, cannot reference the class's type parameters.

**Technical Definition**
A generic class declares type parameters in angle brackets after the class name. Each instance of the generic class is parameterized by the type arguments provided at instantiation. Instance properties, instance methods, and constructors can use the class's type parameters. Static members, however, cannot reference the class's type parameters because static members belong to the class itself, not to instances, and there is only one runtime representation of the class regardless of type arguments. Generic classes can have methods with their own additional type parameters, and can implement generic interfaces.

**Beginner-Friendly Explanation**
A generic class is a class that takes type parameters. For example, `class Box<T> { value: T; }` describes a box that holds a value of any type. When you write `new Box<number>(42)`, you get a box with `value: number`. The type parameter `T` can be used in instance properties, methods, and constructors. But there's a key limitation: static members cannot use the class's type parameters. This is because static members are shared across all instances, and there's only one version of a static member at runtime, regardless of what type arguments you use.

### Purposes

- To create reusable class-based data structures that work with multiple types.
- To provide type-safe containers, collections, and wrappers.
- To implement generic algorithms and patterns as classes.
- To enable type-safe instance methods and properties.
- To support generic class hierarchies and interfaces.

### Syntax Rules and Structure

**General Syntax: Generic Class**

```typescript
class ClassName<T> {
  property: T;
  constructor(value: T) {
    this.property = value;
  }
  method(param: T): T {
    return param;
  }
}
```

**Component Breakdown**
- `<T>`: Type parameter declared after the class name.
- `property: T`: Instance property using the type parameter.
- `constructor(value: T)`: Constructor using the type parameter.
- `method(param: T): T`: Method using the type parameter.

**General Syntax: Generic Class with Multiple Parameters**

```typescript
class Pair<T, U> {
  constructor(public first: T, public second: U) {}
}
```

**Component Breakdown**
- `<T, U>`: Multiple type parameters.

**General Syntax: Generic Class Implementing Generic Interface**

```typescript
class Repository<T extends Entity> implements IRepository<T> {
  findById(id: number): T | undefined { /* ... */ }
}
```

**Component Breakdown**
- The class implements a generic interface with the same type parameter.

**Syntax Rules**

- Type parameters are declared in angle brackets after the class name.
- Type parameters are in scope throughout the class body (instance members only).
- Static members cannot reference the class's type parameters.
- Generic classes can have methods with their own type parameters.
- Generic classes can implement generic interfaces.
- Generic classes can extend other generic classes.
- Type parameters can be constrained with `extends`.
- Type parameters can have default types.

**Constraints and Limitations**

- Static members cannot reference class type parameters.
- Type parameters are erased at runtime; no runtime type information is available.
- Generic classes cannot be used with `new T()` (type parameters are not constructors).
- Complex generic class hierarchies can be difficult to maintain.
- Type inference for generic classes is limited compared to generic functions.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Generic Class

```typescript
// Step 1: Define a generic class.
class Box<T> {
  private _value: T;

  constructor(value: T) {
    this._value = value;
  }

  get value(): T {
    return this._value;
  }

  map<U>(fn: (value: T) => U): Box<U> {
    return new Box(fn(this._value));
  }
}

// Step 2: Create instances with different types.
const numberBox = new Box<number>(42);
const stringBox = new Box<string>("hello");
const objectBox = new Box({ name: "Alice", age: 30 });

// Step 3: Access properties and methods.
console.log(numberBox.value);   // 42
console.log(stringBox.value);   // "hello"
console.log(objectBox.value);   // { name: 'Alice', age: 30 }

// Step 4: Transform with generic method.
const doubled = numberBox.map((n) => n * 2);
console.log(doubled.value);  // 84

const upper = stringBox.map((s) => s.toUpperCase());
console.log(upper.value);    // "HELLO"

// Step 5: Type safety is preserved.
const n: number = numberBox.value;      // ✅
const s: string = stringBox.value;      // ✅
// const wrong: string = numberBox.value; // ❌ Error.
```

**Expected Output:**
```
42
hello
{ name: 'Alice', age: 30 }
84
HELLO
```

**Why This Output Occurs:** The `Box<T>` class is generic over its value type. Each instance has a specific type (`Box<number>`, `Box<string>`, etc.). The `map<U>` method transforms the box's value type from `T` to `U`, preserving type safety.

#### Example 2: Generic Class with Constraint and Static Limitation

```typescript
// Step 1: Define a generic class with a constraint.
interface Entity {
  id: number;
}

class Repository<T extends Entity> {
  private items: Map<number, T> = new Map();

  save(entity: T): void {
    this.items.set(entity.id, entity);
  }

  findById(id: number): T | undefined {
    return this.items.get(id);
  }

  // Static member — cannot reference T.
  // static defaultEntity: T;  // ❌ Error: Static members cannot reference class type parameters.

  // Workaround: use a separate type parameter for static methods.
  static create<U extends Entity>(entity: U): U {
    return entity;
  }
}

// Step 2: Use the generic class.
interface User extends Entity {
  name: string;
  email: string;
}

const userRepo = new Repository<User>();
userRepo.save({ id: 1, name: "Alice", email: "alice@example.com" });
console.log(userRepo.findById(1));  // { id: 1, name: 'Alice', email: 'alice@example.com' }

// Step 3: Static method with its own type parameter.
const user = Repository.create<User>({ id: 2, name: "Bob", email: "bob@example.com" });
console.log(user);  // { id: 2, name: 'Bob', email: 'bob@example.com' }
```

**Expected Output:**
```
{ id: 1, name: 'Alice', email: 'alice@example.com' }
{ id: 2, name: 'Bob', email: 'bob@example.com' }
```

**Why This Output Occurs:** The `Repository<T extends Entity>` class constrains `T` to types with an `id` property. Static members cannot reference `T`, so `static defaultEntity: T` is a compile error. The workaround is to use a separate type parameter for static methods (`static create<U extends Entity>`), which works because static methods have their own scope.

### Real-World Cases

**Case 1: Collection Classes**
Custom collections (`List<T>`, `Stack<T>`, `Queue<T>`) use generics to provide type-safe storage and retrieval.

**Case 2: Result Types**
`Result<T, E>` classes model success/failure outcomes with type-safe value and error access.

**Case 3: Builder Patterns**
Generic builders (`QueryBuilder<T>`, `FormBuilder<T>`) build typed objects step by step.

**Case 4: Caching Layers**
Generic cache classes (`Cache<K, V>`) store key-value pairs with type safety.

**Case 5: Dependency Injection Containers**
Generic DI containers (`Container<T>`) resolve dependencies with type safety.


## 5. Generic Type Aliases and Generic Parameter Scope

### Definitions

**Core Definition**
A generic type alias is a type alias declaration that declares one or more type parameters. The type parameters are in scope throughout the aliased type expression. Type aliases can represent any type—including unions, intersections, primitives, and tuples—making generic type aliases more flexible than generic interfaces.

**Technical Definition**
A generic type alias declares type parameters in angle brackets after the alias name. The type parameters are in scope throughout the aliased type expression and can be used anywhere within it. Unlike interfaces, type aliases can represent non-object types (unions, primitives, tuples, conditional types), and generic type aliases can be recursive. The scope of a type parameter is the entire type alias declaration. Type parameters of a generic type alias are referenced in type references, and writing a reference to a generic type alias instantiates the aliased type with the given type arguments.

**Beginner-Friendly Explanation**
A generic type alias is a type alias that takes type parameters. For example, `type Pair<T> = [T, T]` defines a tuple of two values of the same type. `Pair<number>` gives `[number, number]`. Type aliases are more flexible than interfaces because they can represent unions, primitives, and tuples—not just object shapes. The type parameter scope is the entire alias declaration, so `T` can be used anywhere in the type expression. This makes generic type aliases essential for type-level programming.

### Purposes

- To create reusable type expressions that work with multiple types.
- To define generic tuples, unions, and function types.
- To enable type-level programming with conditional and mapped types.
- To provide concise names for complex generic types.
- To support recursive type definitions with type parameters.

### Syntax Rules and Structure

**General Syntax: Generic Type Alias**

```typescript
type AliasName<T> = TypeExpression;
```

**Component Breakdown**
- `<T>`: Type parameter declared after the alias name.
- `TypeExpression`: Any type expression using `T`.

**General Syntax: Generic Tuple Alias**

```typescript
type Pair<T> = [first: T, second: T];
```

**Component Breakdown**
- The alias defines a tuple with two elements of type `T`.

**General Syntax: Generic Union Alias**

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };
```

**Component Breakdown**
- The alias defines a union with type parameters for data and error.

**General Syntax: Recursive Generic Type Alias**

```typescript
type Tree<T> = {
  value: T;
  children: Tree<T>[];
};
```

**Component Breakdown**
- The alias references itself recursively with the type parameter.

**Syntax Rules**

- Type parameters are declared in angle brackets after the alias name.
- Type parameters are in scope throughout the aliased type expression.
- Type aliases can represent any type: unions, intersections, primitives, tuples, objects.
- Type aliases can be recursive.
- Type parameters can be constrained with `extends`.
- Type parameters can have default types with `=`.
- Generic type aliases can be used in any type position.

**Constraints and Limitations**

- Type parameters are erased at runtime.
- Recursive type aliases must be "guarded" by an object type, array type, or tuple type.
- Type aliases cannot participate in declaration merging.
- Type aliases cannot be extended with `extends` (only intersected).
- Very complex generic type aliases can be difficult to read.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Generic Tuple Alias

```typescript
// Step 1: Define a generic tuple alias.
type Pair<T> = [first: T, second: T];
type Triple<T> = [first: T, second: T, third: T];

// Step 2: Use the aliases.
const numberPair: Pair<number> = [1, 2];
const stringPair: Pair<string> = ["a", "b"];
const mixedTriple: Triple<string | number> = [1, "a", 2];

console.log(numberPair);   // [1, 2]
console.log(stringPair);   // ["a", "b"]
console.log(mixedTriple);  // [1, "a", 2]

// Step 3: Type safety is preserved.
const firstNum: number = numberPair[0];  // ✅
const firstStr: string = stringPair[0];  // ✅

// Step 4: Generic tuple alias in functions.
function swap<T, U>(pair: [T, U]): [U, T] {
  return [pair[1], pair[0]];
}

const swapped = swap(["hello", 42]);
console.log(swapped);  // [42, "hello"]
```

**Expected Output:**
```
[ 1, 2 ]
[ 'a', 'b' ]
[ 1, 'a', 2 ]
[ 42, 'hello' ]
```

**Why This Output Occurs:** The `Pair<T>` alias defines a tuple of two `T` values. The `swap<T, U>` function uses its own type parameters to swap the tuple elements, preserving the types.

#### Example 2: Recursive Generic Type Alias

```typescript
// Step 1: Define a recursive generic type alias for a tree.
type Tree<T> = {
  value: T;
  children: Tree<T>[];
};

// Step 2: Create a tree of numbers.
const numberTree: Tree<number> = {
  value: 1,
  children: [
    { value: 2, children: [] },
    { value: 3, children: [{ value: 4, children: [] }] },
  ],
};

// Step 3: Create a tree of strings.
const stringTree: Tree<string> = {
  value: "root",
  children: [
    { value: "child1", children: [] },
    { value: "child2", children: [] },
  ],
};

// Step 4: Recursive processing.
function sumTree(tree: Tree<number>): number {
  return tree.value + tree.children.reduce((sum, child) => sum + sumTree(child), 0);
}

console.log(sumTree(numberTree));  // 10

// Step 5: Generic type alias with constraint.
type Identifiable = { id: number };
type EntityTree<T extends Identifiable> = {
  entity: T;
  children: EntityTree<T>[];
};

const entityTree: EntityTree<{ id: number; name: string }> = {
  entity: { id: 1, name: "Root" },
  children: [
    { entity: { id: 2, name: "Child" }, children: [] },
  ],
};

console.log(entityTree.entity.name);  // "Root"
```

**Expected Output:**
```
10
Root
```

**Why This Output Occurs:** The `Tree<T>` alias recursively defines a tree structure where each node has a value of type `T` and an array of child trees. The `EntityTree<T extends Identifiable>` alias adds a constraint requiring an `id` property. The recursive type works because the self-reference is guarded by the object type.

### Real-World Cases

**Case 1: Utility Types**
`Partial<T>`, `Required<T>`, `Readonly<T>`, and `Record<K, V>` are generic type aliases.

**Case 2: Result and Either Types**
`Result<T, E>` and `Either<L, R>` are generic type aliases for error handling.

**Case 3: Recursive Data Structures**
Trees, linked lists, and JSON types use recursive generic type aliases.

**Case 4: Function Type Aliases**
`type Mapper<T, U> = (item: T) => U` is a generic function type alias.

**Case 5: Conditional Type Aliases**
`type IsArray<T> = T extends any[] ? true : false` is a generic conditional type alias.


## 6. Generic Arrow Function Syntax Constraints and the `<T,>` / `T extends unknown` JSX Parsing Workaround

### Definitions

**Core Definition**
Generic arrow functions in TypeScript have a syntactic ambiguity in `.tsx` files: the `<T>` syntax for type parameters conflicts with JSX opening tags. Two workarounds exist: adding a trailing comma (`<T,>`) or constraining the type parameter (`<T extends unknown>`). These disambiguate the generic arrow function from JSX.

**Technical Definition**
In `.tsx` files, the parser cannot distinguish between `<T>(value: T) => value` (a generic arrow function) and `<T>(value: T) => value</T>` (JSX). The trailing comma workaround `<T,>` provides a parse hint that the angle brackets denote type parameters, not JSX. The constraint workaround `<T extends unknown>` provides an explicit type constraint that disambiguates. Both workarounds are semantically equivalent to `<T>` in `.ts` files. The trailing comma is the most concise; the constraint is more explicit and works in all contexts. A third option is to use a named function declaration instead of an arrow function.

**Beginner-Friendly Explanation**
In `.tsx` files (TypeScript + JSX), writing a generic arrow function like `<T>(value: T) => value` confuses the parser—it thinks you're starting a JSX element. To fix this, you can either add a comma: `<T,>(value: T) => value`, or add a constraint: `<T extends unknown>(value: T) => value`. Both tell TypeScript "this is a generic function, not JSX." The trailing comma is shorter; the constraint is more explicit. You only need this workaround in `.tsx` files—in `.ts` files, `<T>` works fine. If you don't want to deal with it, use a named function instead of an arrow function.

### Purposes

- To write generic arrow functions in `.tsx` files.
- To disambiguate generic syntax from JSX syntax.
- To enable generic React components and hooks with arrow function syntax.
- To understand the parsing constraints of `.tsx` files.
- To choose the appropriate workaround for the context.

### Syntax Rules and Structure

**General Syntax: Trailing Comma Workaround**

```typescript
const identity = <T,>(value: T): T => value;
```

**Component Breakdown**
- `<T,>`: The trailing comma disambiguates from JSX.

**General Syntax: Constraint Workaround**

```typescript
const identity = <T extends unknown>(value: T): T => value;
```

**Component Breakdown**
- `<T extends unknown>`: The constraint disambiguates from JSX.

**General Syntax: Named Function (Alternative)**

```typescript
function identity<T>(value: T): T {
  return value;
}
```

**Component Breakdown**
- Named functions do not have the JSX ambiguity.

**General Syntax: Function Expression (Alternative)**

```typescript
const identity = function <T>(value: T): T {
  return value;
};
```

**Component Breakdown**
- Function expressions with `function` keyword also work.

**Syntax Rules**

- In `.tsx` files, `<T>` alone is ambiguous with JSX.
- The trailing comma `<T,>` disambiguates.
- The constraint `<T extends unknown>` disambiguates.
- Named functions and function expressions do not have the ambiguity.
- The workarounds are only needed in `.tsx` files.
- In `.ts` files, `<T>` works without modification.
- Multiple type parameters also need the workaround: `<T, U>` works, but `<T,>` is safer.

**Constraints and Limitations**

- The trailing comma is a syntax hack; it has no semantic meaning.
- The constraint `<T extends unknown>` is semantically equivalent to `<T>`.
- Some tools and linters may flag the trailing comma as unnecessary.
- The workaround is specific to `.tsx` files; in `.ts` files, no workaround is needed.
- The workaround does not apply to generic function declarations (only arrow functions).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Trailing Comma Workaround

```typescript
// Step 1: Define a generic arrow function in a .tsx file.
// The trailing comma disambiguates from JSX.
const identity = <T,>(value: T): T => value;

// Step 2: Use the function.
const num = identity(42);
const str = identity("hello");

console.log(num);  // 42
console.log(str);  // "hello"

// Step 3: Type safety is preserved.
const n: number = identity(100);      // ✅
const s: string = identity("world");  // ✅

// Step 4: Multiple type parameters.
const pair = <T, U>(first: T, second: U): [T, U] => [first, second];
const result = pair("hello", 42);
console.log(result);  // ["hello", 42]

// Step 5: Without the comma, this would be a JSX error in .tsx.
// const wrong = <T>(value: T): T => value;
// ❌ Error: JSX element 'T' has no corresponding closing tag.
```

**Expected Output:**
```
42
hello
[ 'hello', 42 ]
```

**Why This Output Occurs:** The trailing comma in `<T,>` tells the TypeScript parser that the angle brackets denote type parameters, not JSX. The generic arrow function works normally, preserving type safety. Without the comma, the parser would interpret `<T>` as a JSX opening tag.

#### Example 2: Constraint Workaround

```typescript
// Step 1: Define a generic arrow function with a constraint.
const identity = <T extends unknown>(value: T): T => value;

// Step 2: Use the function.
const num = identity(42);
const str = identity("hello");

console.log(num);  // 42
console.log(str);  // "hello"

// Step 3: Constrained generic arrow function.
const logLength = <T extends { length: number }>(value: T): T => {
  console.log(`Length: ${value.length}`);
  return value;
};

logLength("hello");     // "Length: 5"
logLength([1, 2, 3]);   // "Length: 3"

// Step 4: Named function alternative (no workaround needed).
function identityNamed<T>(value: T): T {
  return value;
}

const result = identityNamed("hello");
console.log(result);  // "hello"
```

**Expected Output:**
```
42
hello
Length: 5
Length: 3
hello
```

**Why This Output Occurs:** The constraint `<T extends unknown>` is semantically identical to `<T>` but disambiguates from JSX. The `logLength` function uses a more specific constraint (`{ length: number }`) to enable `.length` access. The named function has no ambiguity and needs no workaround.

### Real-World Cases

**Case 1: Generic React Components**
React components with generic props use the workaround to write generic arrow function components in `.tsx` files.

**Case 2: Generic Hooks**
Custom React hooks with generic state types use the workaround for arrow function declarations.

**Case 3: Utility Libraries**
Libraries distributed as `.tsx` files use the workaround to provide generic utility functions.

**Case 4: Test Utilities**
Test helpers with generic assertions use the workaround for arrow function syntax.

---

## References

- TypeScript Handbook: Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook: Generic Parameter Defaults — https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-parameter-defaults
- TypeScript Language Specification: Type Aliases — https://chromium.googlesource.com/external/github.com/kythe/kythe/+/7c2c91248ad9bc8d22dfaa00764c6381af512113/third_party/typescript/doc/spec.md
- TypeScript Language Specification: Generic Types — https://github.com/microsoft/TypeScript/blob/main/doc/spec.md
- TypeScript Deep Dive: Generics — https://basarat.gitbook.io/typescript/type-system/generics
- TypeScript Deep Dive: Arrow Generics — https://basarat.gitbook.io/typescript/type-system/generics#arrow-generics
- React + TypeScript Cheatsheets: Generic Components — https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#generic-components
- Total TypeScript: Generic Constraints — https://www.totaltypescript.com/workshops/typescript-pro-essentials/the-utils-folder/type-parameter-constraints-with-generic-functions/solution
- Steve Kinney: TypeScript Generics Deep Dive — https://stevekinney.com/courses/react-typescript/typescript-generics-deep-dive
- Stack Overflow: Generic Arrow Functions in TSX — https://stackoverflow.com/questions/41112313/generic-arrow-functions-in-tsx
- Stack Overflow: Static Attribute of Generic Type — https://stackoverflow.com/questions/41089854/typescript-access-static-attribute-of-generic-type
- MDN: JavaScript Classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- MDN: Arrow Function Expressions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions