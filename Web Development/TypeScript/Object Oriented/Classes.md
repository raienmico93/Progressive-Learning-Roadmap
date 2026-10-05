# TypeScript Classes & Instantiation: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A class in TypeScript is a blueprint for creating objects that share common properties and methods. TypeScript extends JavaScript's class syntax with static type checking, access modifiers, parameter properties, and additional features that improve encapsulation and type safety.

**Technical Definition**
A TypeScript class is a value-space and type-space construct that defines a constructor, instance members (fields, methods, accessors), and static members. Classes create both a constructor function (value) and an instance type (type). TypeScript augments JavaScript classes with type annotations, access modifiers (`public`, `private`, `protected`), the `readonly` modifier, parameter properties (constructor parameter shorthand), abstract classes, and interface implementation checking. TypeScript's class system is structurally typed for compatibility, except when private or protected members introduce nominal behavior. The `this` type, `this` parameter, and polymorphic `this` return types enable fluent interfaces and inheritance-safe APIs.

**Beginner-Friendly Explanation**
A class is a template for making objects. If you want to create many user objects with the same properties and behaviors, you write a `User` class once and then create instances with `new User(...)`. TypeScript adds extra safety features to classes: you can mark properties as `private` (only accessible inside the class) or `readonly` (can't be changed after creation), and TypeScript checks that your class implements any interfaces it claims to. Classes are the foundation of object-oriented programming in TypeScript, though modern TypeScript developers often combine classes with functional patterns.

### Key Characteristics

- **Blueprint for objects**: Classes define the structure and behavior of instances.
- **Dual nature**: A class creates both a constructor function (value) and an instance type (type).
- **Access modifiers**: `public`, `private`, and `protected` control visibility.
- **Parameter properties**: Constructor parameters can declare and initialize class fields simultaneously.
- **Hard private fields**: JavaScript's `#private` syntax provides true runtime privacy.
- **Accessors**: `get`/`set` accessors enable computed properties.
- **Static members**: Properties, methods, and initialization blocks belong to the class itself.
- **Structural typing**: Class compatibility is structural except with private/protected members.
- **`this` polymorphism**: The `this` type enables inheritance-safe fluent APIs.

### Prerequisites

- Basic knowledge of JavaScript classes (constructor, methods, `new`)
- Familiarity with TypeScript interfaces and object types
- Understanding of TypeScript access modifiers and readonly properties
- Familiarity with inheritance and the `extends` keyword

### Related Programming Areas

- **Object-Oriented Programming**: Classes are the foundation of OOP
- **Design Patterns**: Factory, singleton, observer, and strategy patterns use classes
- **Dependency Injection**: Classes are often injected as dependencies
- **Interface Implementation**: Classes implement interfaces to fulfill contracts
- **JavaScript Prototypes**: TypeScript classes compile to JavaScript's prototype-based system

### Core Concepts / Features

1. Class Field Declaration and Structural Inference
2. Constructors and Initialization
3. Instance Methods and `this` Binding Behaviors
4. Parameter Properties (`constructor(private name: string)`)
5. TypeScript Access Modifiers (`public`, `private`, `protected`)
6. JavaScript Hard Private Fields (`#private`)
7. Getters and Setters (`get` / `set` accessors)
8. `readonly` Properties in Classes
9. Static Members (Properties, Blocks, and Methods)


## 1. Class Field Declaration and Structural Inference

### Definitions

**Core Definition**
Class field declarations define the properties that each instance of a class will have. Fields can be declared with type annotations or inferred from initializers. TypeScript infers field types from their initial values, and field declarations participate in the class's structural type.

**Technical Definition**
Class fields in TypeScript are declared either with explicit type annotations (`field: Type;`) or with initializers (`field = value;`). The field's type is the annotation if provided, otherwise the widened type of the initializer. Without `useDefineForClassFields` (a `tsconfig` option tied to the target), fields declared without initializers are type-only and emit nothing. With `useDefineForClassFields: true` (default for ES2022+), fields are emitted as `Object.defineProperty` calls, and declared-but-uninitialized fields are set to `undefined`. Field declarations contribute to the class's instance type and are structurally checked when instances are assigned to interface types.

**Beginner-Friendly Explanation**
Class fields are the properties each object will have. When you write `class User { name: string = "Alice"; age: number; }`, you're saying every `User` instance has a `name` (defaulting to `"Alice"`) and an `age`. TypeScript can infer field types from initializers: `name = "Alice"` makes `name` a `string`. Fields are part of the class's type, so TypeScript knows what properties exist on instances. The `useDefineForClassFields` option changes some subtle behavior about how fields are initialized, especially for fields without initializers.

### Purposes

- To declare the properties that each instance of a class will have.
- To provide default values for fields at instance creation.
- To enable type inference for field types from initializers.
- To participate in structural typing so instances match interface types.
- To control field initialization behavior with `useDefineForClassFields`.

### Syntax Rules and Structure

**General Syntax: Field Declaration with Type Annotation**

```typescript
class ClassName {
  fieldName: FieldType;
  initializedField: FieldType = initialValue;
}
```

**Component Breakdown**
- `fieldName: FieldType`: A field with an explicit type annotation (no initializer).
- `initializedField: FieldType = initialValue`: A field with an initializer.

**General Syntax: Field Declaration with Inferred Type**

```typescript
class ClassName {
  fieldName = initialValue;  // Type inferred from initialValue
}
```

**Component Breakdown**
- The field's type is the widened type of `initialValue`.

**General Syntax: Readonly and Access-Modified Fields**

```typescript
class ClassName {
  readonly readonlyField: Type = value;
  private privateField: Type = value;
  protected protectedField: Type = value;
  static staticField: Type = value;
}
```

**Component Breakdown**
- Modifiers can be combined on field declarations.

**Syntax Rules**

- Fields can have explicit type annotations or rely on inference from initializers.
- Fields without initializers and without type annotations infer as `any` (with `noImplicitAny` errors).
- The `useDefineForClassFields` option (default for ES2022+) changes field initialization semantics.
- Fields with `declare` modifier emit no code (used with `useDefineForClassFields` to avoid shadowing).
- Fields can be `readonly`, `public`, `private`, `protected`, or `static`.
- Field initializers run in declaration order during construction.
- Fields are part of the instance type for structural typing.

**Constraints and Limitations**

- Uninitialized fields without type annotations infer as `any` under `noImplicitAny`.
- With `useDefineForClassFields: true`, declared-but-uninitialized fields are set to `undefined`, potentially shadowing prototype properties.
- The `declare` modifier is needed to declare fields that exist on the prototype without emitting code.
- Field initializers cannot reference `this` before `super()` in derived classes.
- Field initialization order matters when initializers depend on each other.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Field Declarations with Inference

```typescript
// Step 1: Declare a class with various field declarations.
class Product {
  id: number;                    // Explicit type, no initializer
  name = "Unnamed Product";      // Inferred type: string
  price = 0;                     // Inferred type: number
  tags: string[] = [];           // Explicit type with initializer
  inStock = true;                // Inferred type: boolean

  constructor(id: number) {
    this.id = id;                // Initialize the uninitialized field
  }
}

// Step 2: Create instances.
const product = new Product(1);
console.log(product);
// Product { id: 1, name: "Unnamed Product", price: 0, tags: [], inStock: true }

// Step 3: Field types are enforced.
product.name = "Laptop";        // ✅ Allowed
// product.price = "expensive"; // ❌ Error: Type 'string' is not assignable to type 'number'.

// Step 4: Fields participate in structural typing.
interface ProductLike {
  id: number;
  name: string;
}

const productLike: ProductLike = product;  // ✅ Allowed — Product has id and name
console.log(productLike.name);  // "Laptop"
```

**Expected Output:**
```
Product { id: 1, name: 'Unnamed Product', price: 0, tags: [], inStock: true }
Laptop
```

**Why This Output Occurs:** TypeScript infers field types from initializers (`name: string`, `price: number`) and uses the explicit annotation for `id`. The `Product` instance satisfies the `ProductLike` interface because it has compatible `id` and `name` properties (structural typing).

#### Example 2: `useDefineForClassFields` Behavior

```typescript
// Step 1: Class with a declared field (no initializer).
class Example {
  declaredField: string;  // No initializer
  initializedField = "hello";
}

// Step 2: With useDefineForClassFields: false (legacy behavior),
// declaredField is type-only and emits nothing.
// The instance has no own property "declaredField" until assigned.

// Step 3: With useDefineForClassFields: true (ES2022 default),
// declaredField is emitted as Object.defineProperty(this, "declaredField", undefined).
// The instance has an own property "declaredField" set to undefined.

// Step 4: With the "declare" modifier, no code is emitted regardless.
class ExampleWithDeclare {
  declare declaredField: string;  // No runtime effect
  initializedField = "hello";
}

const ex = new ExampleWithDeclare();
console.log(ex.initializedField);  // "hello"
console.log("declaredField" in ex);  // false (declare emits nothing)
```

**Expected Output:**
```
hello
false
```

**Why This Output Occurs:** The `declare` modifier tells TypeScript that the field exists on the type but should not emit any code. With `useDefineForClassFields: true`, `declaredField` would otherwise be defined as `undefined` on the instance. Using `declare` avoids this, which is important when extending classes that have prototype properties.

### Real-World Cases

**Case 1: Domain Entities**
Domain entities (User, Product, Order) declare fields with types and defaults, providing a clear structure for instance data.

**Case 2: API Client Classes**
API client classes declare fields for configuration (baseUrl, timeout, retries) with sensible defaults, enabling flexible instantiation.

**Case 3: React Class Components**
Legacy React class components declare fields for state and props, with types ensuring type-safe access.

---

## 2. Constructors and Initialization

### Definitions

**Core Definition**
A constructor is a special method in a class that runs when an instance is created with `new`. The constructor initializes instance fields, performs setup logic, and can accept parameters that configure the instance. In derived classes, the constructor must call `super()` before accessing `this`.

**Technical Definition**
The constructor method (`constructor(...)`) is invoked during instance creation. TypeScript types constructor parameters and checks that `new ClassName(args)` matches the constructor signature. In derived classes, `super(...)` must be called before any `this` access; TypeScript enforces this. Constructor overloads can be declared with multiple signature declarations followed by a single implementation. Constructor return type is always the class instance type (or a subtype). Parameter properties (covered separately) allow declaring and initializing fields directly in the constructor parameter list.

**Beginner-Friendly Explanation**
A constructor is a special function that runs when you create a new object from a class. It's where you set up the object's initial state—assigning values to fields, validating inputs, and performing any setup. When you write `new User("Alice", 30)`, the constructor receives `"Alice"` and `30` and uses them to initialize the `User` instance. In classes that extend other classes, the constructor must call `super()` first to initialize the parent class before using `this`.

### Purposes

- To initialize instance fields when an object is created.
- To validate or transform constructor arguments before assignment.
- To perform setup logic (e.g., opening connections, registering listeners).
- To enforce invariants at object creation time.
- To support overloaded constructor signatures for flexible instantiation.

### Syntax Rules and Structure

**General Syntax: Constructor Declaration**

```typescript
class ClassName {
  field: Type;

  constructor(param1: ParamType1, param2: ParamType2) {
    this.field = param1;
    // initialization logic
  }
}
```

**Component Breakdown**
- `constructor(...)`: The constructor method (no return type annotation).
- Parameters are typed like function parameters.
- `this.field = param1`: Field initialization.

**General Syntax: Derived Class Constructor**

```typescript
class ChildClass extends ParentClass {
  field: Type;

  constructor(param: ParamType) {
    super(parentArg);  // Must call super() first
    this.field = param;
  }
}
```

**Component Breakdown**
- `super(parentArg)`: Calls the parent constructor.
- `this` is only accessible after `super()`.

**General Syntax: Constructor Overloads**

```typescript
class ClassName {
  constructor(value: string);
  constructor(value: number);
  constructor(value: string | number) {
    // implementation
  }
}
```

**Component Breakdown**
- Multiple signature declarations followed by one implementation signature.

**Syntax Rules**

- The constructor method is named `constructor`.
- Constructor parameters are typed like function parameters.
- In derived classes, `super()` must be called before `this` access.
- TypeScript enforces `super()` call order.
- Constructor overloads require one implementation signature.
- Constructors can be `private` or `protected` to prevent direct instantiation.
- A class without a constructor has an implicit no-arg constructor.
- The constructor's return type is always the class instance type.

**Constraints and Limitations**

- Constructors cannot have return type annotations.
- In derived classes, `super()` must be called before any `this` access.
- Constructor overloads must be ordered from most specific to least specific.
- Private constructors prevent `new ClassName()` outside the class (factory pattern).
- Constructors run field initializers before the constructor body (in declaration order).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Constructor

```typescript
// Step 1: Declare a class with a constructor.
class User {
  id: number;
  name: string;
  email: string;

  constructor(id: number, name: string, email: string) {
    this.id = id;
    this.name = name;
    this.email = email;
    console.log(`User ${name} created with ID ${id}`);
  }
}

// Step 2: Create instances.
const alice = new User(1, "Alice", "alice@example.com");
// Output: "User Alice created with ID 1"

const bob = new User(2, "Bob", "bob@example.com");
// Output: "User Bob created with ID 2"

// Step 3: Constructor arguments are type-checked.
// const charlie = new User("3", "Charlie", "charlie@example.com");
// ❌ Error: Argument of type 'string' is not assignable to parameter of type 'number'.

// Step 4: Access initialized fields.
console.log(alice.name);   // "Alice"
console.log(bob.email);    // "bob@example.com"
```

**Expected Output:**
```
User Alice created with ID 1
User Bob created with ID 2
Alice
bob@example.com
```

**Why This Output Occurs:** The constructor receives arguments matching its typed parameters, assigns them to fields, and logs a creation message. TypeScript checks that `new User(...)` arguments match the constructor signature.

#### Example 2: Derived Class with `super()`

```typescript
// Step 1: Define a base class.
class Animal {
  name: string;

  constructor(name: string) {
    this.name = name;
    console.log(`Animal ${name} created`);
  }

  move(): void {
    console.log(`${this.name} moves.`);
  }
}

// Step 2: Define a derived class.
class Dog extends Animal {
  breed: string;

  constructor(name: string, breed: string) {
    super(name);  // Must call super() before accessing this
    this.breed = breed;
    console.log(`Dog ${name} (${breed}) created`);
  }

  bark(): void {
    console.log(`${this.name} barks!`);
  }
}

// Step 3: Create a Dog instance.
const rex = new Dog("Rex", "German Shepherd");
// Output:
// "Animal Rex created"
// "Dog Rex (German Shepherd) created"

// Step 4: Call inherited and own methods.
rex.move();  // "Rex moves."
rex.bark();  // "Rex barks!"
```

**Expected Output:**
```
Animal Rex created
Dog Rex (German Shepherd) created
Rex moves.
Rex barks!
```

**Why This Output Occurs:** The `Dog` constructor calls `super(name)` first, which invokes the `Animal` constructor and logs "Animal Rex created". Then the `Dog` constructor assigns `breed` and logs its own message. `rex.move()` uses the inherited method, and `rex.bark()` uses the Dog-specific method.

### Real-World Cases

**Case 1: API Client Initialization**
API client classes use constructors to accept configuration (base URL, API key, timeout) and initialize internal state like HTTP headers and connection pools.

**Case 2: Domain Entity Creation**
Domain entities use constructors to enforce invariants (e.g., a `Money` class validating that amounts are non-negative) and initialize derived fields.

**Case 3: Factory Pattern**
Classes with private constructors use static factory methods to control instantiation, returning cached or configured instances.

---

## 3. Instance Methods and `this` Binding Behaviors

### Definitions

**Core Definition**
Instance methods are functions defined on a class that operate on individual instances. TypeScript types `this` within methods as the instance type, enabling type-safe access to instance fields and methods. The `this` binding can be explicitly typed using `this` parameters to control method context.

**Technical Definition**
Instance methods are declared using method syntax (`methodName(params): ReturnType { }`). TypeScript infers the type of `this` within methods as the instance type (`this: ClassName`), giving type-safe access to fields and methods. The `this` parameter (a fake first parameter) allows explicit typing of `this`, which is useful for callbacks and event handlers. The polymorphic `this` type (`this: this`) enables fluent APIs and inheritance-safe method chaining. Arrow function class properties capture the lexical `this`, avoiding binding issues.

**Beginner-Friendly Explanation**
Instance methods are functions that belong to each object created from a class. When you call `alice.greet()`, the method runs with `this` referring to `alice`, so `this.name` accesses Alice's name. TypeScript knows that inside a method, `this` is the instance type, so it can catch mistakes like `this.nmae`. The `this` parameter lets you explicitly type `this` for callbacks. Arrow function properties capture `this` from the enclosing scope, which is useful when passing methods as callbacks (e.g., to event listeners).

### Purposes

- To define behavior that operates on instance data.
- To enable type-safe access to instance fields and methods via `this`.
- To support method chaining and fluent APIs with the polymorphic `this` type.
- To control `this` binding for callbacks using arrow function properties.
- To explicitly type `this` in functions extracted from classes.

### Syntax Rules and Structure

**General Syntax: Instance Method**

```typescript
class ClassName {
  field: Type;

  methodName(param: ParamType): ReturnType {
    return this.field;  // `this` is the instance
  }
}
```

**Component Breakdown**
- `methodName(params): ReturnType`: The method declaration.
- `this`: Refers to the instance within the method.

**General Syntax: `this` Parameter**

```typescript
class ClassName {
  methodName(this: ClassName, param: ParamType): ReturnType {
    // `this` is explicitly typed
  }
}
```

**Component Breakdown**
- `this: ClassName`: The fake first parameter explicitly types `this`.

**General Syntax: Polymorphic `this`**

```typescript
class ClassName {
  setValue(value: number): this {
    // ...
    return this;
  }
}
```

**Component Breakdown**
- `: this`: The return type is the polymorphic `this` type.
- Enables method chaining in subclasses.

**General Syntax: Arrow Function Property**

```typescript
class ClassName {
  methodName = (param: ParamType): ReturnType => {
    return this.field;  // Lexically captured `this`
  };
}
```

**Component Breakdown**
- `= (...) => { }`: An arrow function class property captures the lexical `this`.

**Syntax Rules**

- Instance methods use method syntax (`methodName(params): ReturnType { }`).
- `this` inside a method is typed as the instance type.
- The `this` parameter (fake first parameter) explicitly types `this`.
- The polymorphic `this` return type enables inheritance-safe chaining.
- Arrow function properties capture the lexical `this` from the enclosing scope.
- Methods are on the prototype; arrow function properties are on the instance.
- TypeScript's `noImplicitThis` option catches untyped `this` usage.

**Constraints and Limitations**

- Methods extracted from instances lose their `this` binding unless bound or arrow-wrapped.
- Arrow function properties consume more memory (one per instance vs. one on the prototype).
- The `this` parameter cannot be used with arrow functions.
- Polymorphic `this` cannot be used with static methods.
- `this` in static methods refers to the class itself, not an instance.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Instance Methods and `this`

```typescript
// Step 1: Define a class with instance methods.
class Counter {
  count: number = 0;

  increment(): void {
    this.count += 1;  // `this` is the Counter instance
  }

  decrement(): void {
    this.count -= 1;
  }

  getCount(): number {
    return this.count;
  }
}

// Step 2: Create an instance and call methods.
const counter = new Counter();
counter.increment();
counter.increment();
counter.decrement();
console.log(counter.getCount());  // 1

// Step 3: `this` is type-checked.
class BadCounter {
  value: number = 0;
  increment(): void {
    // this.nonExistent += 1;  // ❌ Error: Property 'nonExistent' does not exist.
  }
}
```

**Expected Output:**
```
1
```

**Why This Output Occurs:** Each method call operates on the `counter` instance via `this`. TypeScript knows `this.count` is a number, so `this.count += 1` is type-safe.

#### Example 2: Polymorphic `this` and Arrow Function Properties

```typescript
// Step 1: Define a class with polymorphic `this` return type.
class QueryBuilder {
  protected conditions: string[] = [];

  where(condition: string): this {
    this.conditions.push(condition);
    return this;  // Returns the current instance
  }

  build(): string {
    return this.conditions.length > 0
      ? `WHERE ${this.conditions.join(" AND ")}`
      : "";
  }
}

// Step 2: Method chaining works because of polymorphic `this`.
const query = new QueryBuilder()
  .where("age > 18")
  .where("country = 'US'")
  .build();
console.log(query);  // "WHERE age > 18 AND country = 'US'"

// Step 3: Polymorphic `this` preserves subclass types.
class AdvancedQueryBuilder extends QueryBuilder {
  orderBy(field: string): this {
    console.log(`Order by ${field}`);
    return this;
  }
}

const advanced = new AdvancedQueryBuilder()
  .where("active = true")
  .orderBy("name")  // ✅ Available because `this` is AdvancedQueryBuilder
  .build();
console.log(advanced);  // "WHERE active = true"

// Step 4: Arrow function property captures lexical `this`.
class Timer {
  seconds = 0;

  start = (): void => {
    // `this` is lexically captured — works as a callback
    setInterval(() => {
      this.seconds += 1;
    }, 1000);
  };
}
```

**Expected Output:**
```
WHERE age > 18 AND country = 'US'
Order by name
WHERE active = true
```

**Why This Output Occurs:** The polymorphic `this` return type ensures that chaining returns the correct subclass type. In `AdvancedQueryBuilder`, `orderBy` is available after `where` because `this` is the subclass type. The arrow function property in `Timer` captures `this` lexically, so it works correctly when passed as a callback.

### Real-World Cases

**Case 1: Fluent APIs and Builders**
Query builders, HTTP clients, and configuration builders use polymorphic `this` to enable method chaining with correct subclass types.

**Case 2: Event Handlers**
Arrow function properties are used for event handlers in React class components and DOM event listeners to preserve `this` binding.

**Case 3: Callback Passing**
When passing methods as callbacks (e.g., to `Array.map` or `setTimeout`), arrow function properties avoid `this` binding issues.

---

## 4. Parameter Properties (`constructor(private name: string)`)

### Definitions

**Core Definition**
Parameter properties are a TypeScript shorthand that allows constructor parameters to be declared as class fields simultaneously. By adding an access modifier (`public`, `private`, `protected`) or `readonly` to a constructor parameter, TypeScript generates a field with the same name and initializes it with the parameter value.

**Technical Definition**
Parameter properties combine field declaration and constructor parameter declaration in a single syntax. When a constructor parameter has an access modifier (`public`, `private`, `protected`), `readonly`, or both, TypeScript emits a field declaration and assigns the parameter value to `this.paramName` at the start of the constructor body. Parameter properties must appear before regular parameters in the constructor. They reduce boilerplate for the common pattern of accepting constructor arguments and assigning them to fields.

**Beginner-Friendly Explanation**
Parameter properties are a shortcut for a common pattern. Instead of writing a field declaration and a constructor that assigns to it, you write the modifier directly on the constructor parameter. For example, `constructor(private name: string) {}` automatically creates a `private name` field and assigns the constructor argument to it. It's a compact way to write classes with dependency injection or configuration parameters. You can use `public`, `private`, `protected`, `readonly`, or combinations.

### Purposes

- To reduce boilerplate by combining field declaration and constructor parameter declaration.
- To implement dependency injection patterns concisely.
- To create classes with configuration parameters that become instance fields.
- To make class definitions shorter and more readable.
- To enforce access control on constructor-injected fields.

### Syntax Rules and Structure

**General Syntax: Parameter Property**

```typescript
class ClassName {
  constructor(
    public publicField: Type1,
    private privateField: Type2,
    protected protectedField: Type3,
    readonly readonlyField: Type4
  ) {}
}
```

**Component Breakdown**
- `public`, `private`, `protected`, `readonly`: Modifiers on constructor parameters.
- Each parameter becomes a field with the same name and the modifier applied.
- The parameter value is assigned to `this.paramName`.

**General Syntax: Combined Modifiers**

```typescript
class ClassName {
  constructor(
    private readonly name: string,
    public readonly id: number
  ) {}
}
```

**Component Breakdown**
- `private readonly`: The field is private and readonly.
- `public readonly`: The field is public and readonly.

**Syntax Rules**

- Parameter properties use access modifiers or `readonly` on constructor parameters.
- Parameter properties must appear before regular parameters.
- The generated field has the same name as the parameter.
- The parameter value is assigned to the field at the start of the constructor.
- Parameter properties can be combined with regular parameters.
- The constructor body can access parameter properties via `this`.
- Parameter properties are public by default if no modifier is specified (they are not parameter properties unless a modifier is present).

**Constraints and Limitations**

- Parameter properties must come before regular parameters.
- Modifiers cannot be applied to rest parameters.
- Parameter properties cannot be used with destructuring parameters.
- The generated code adds `this.paramName = paramName` at the top of the constructor body.
- Parameter properties cannot be conditionally initialized (the assignment always happens).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Parameter Properties

```typescript
// Step 1: Define a class using parameter properties.
class User {
  constructor(
    public id: number,
    public name: string,
    private email: string,
    readonly createdAt: Date = new Date()
  ) {}

  getEmail(): string {
    return this.email;  // Accessible within the class
  }

  describe(): string {
    return `${this.name} (${this.getEmail()})`;
  }
}

// Step 2: Create instances.
const alice = new User(1, "Alice", "alice@example.com");
console.log(alice.id);           // 1 (public)
console.log(alice.name);         // "Alice" (public)
// console.log(alice.email);     // ❌ Error: 'email' is private.
console.log(alice.getEmail());   // "alice@example.com"
console.log(alice.describe());   // "Alice (alice@example.com)"

// Step 3: Readonly parameter property cannot be reassigned.
// alice.createdAt = new Date();  // ❌ Error: Cannot assign to 'createdAt'.
console.log(alice.createdAt instanceof Date);  // true

// Step 4: Equivalent verbose version (for comparison).
class UserVerbose {
  public id: number;
  public name: string;
  private email: string;
  readonly createdAt: Date;

  constructor(id: number, name: string, email: string, createdAt: Date = new Date()) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.createdAt = createdAt;
  }

  getEmail(): string { return this.email; }
}
```

**Expected Output:**
```
1
Alice
alice@example.com
Alice (alice@example.com)
true
```

**Why This Output Occurs:** The parameter properties `public id`, `public name`, `private email`, and `readonly createdAt` generate fields with the same names and assign the constructor arguments. The `private email` field is only accessible within the class (via `getEmail`). The `readonly createdAt` field cannot be reassigned.

#### Example 2: Dependency Injection with Parameter Properties

```typescript
// Step 1: Define service interfaces.
interface Logger {
  log(message: string): void;
}

interface Database {
  query(sql: string): unknown[];
}

// Step 2: Define a class with parameter properties for dependency injection.
class UserService {
  constructor(
    private readonly logger: Logger,
    private readonly db: Database,
    private readonly tableName: string = "users"
  ) {}

  getUser(id: number): unknown {
    this.logger.log(`Fetching user ${id} from ${this.tableName}`);
    const results = this.db.query(`SELECT * FROM ${this.tableName} WHERE id = ${id}`);
    return results[0];
  }
}

// Step 3: Create mock implementations.
const consoleLogger: Logger = {
  log: (msg) => console.log(`[LOG] ${msg}`),
};

const mockDb: Database = {
  query: (sql) => {
    console.log(`[DB] ${sql}`);
    return [{ id: 1, name: "Alice" }];
  },
};

// Step 4: Inject dependencies.
const service = new UserService(consoleLogger, mockDb);
const user = service.getUser(1);
console.log(user);  // { id: 1, name: "Alice" }
```

**Expected Output:**
```
[LOG] Fetching user 1 from users
[DB] SELECT * FROM users WHERE id = 1
{ id: 1, name: 'Alice' }
```

**Why This Output Occurs:** The parameter properties `private readonly logger`, `private readonly db`, and `private readonly tableName` automatically create and initialize fields. The `UserService` class receives its dependencies through the constructor, enabling dependency injection without explicit field declarations.

### Real-World Cases

**Case 1: Dependency Injection**
Services, repositories, and controllers use parameter properties to receive dependencies concisely, especially in frameworks like NestJS and Angular.

**Case 2: Value Objects**
Value objects (Money, DateRange, Coordinates) use parameter properties to encapsulate their components with appropriate access modifiers.

**Case 3: Configuration Wrappers**
Classes that wrap configuration objects use parameter properties to expose configuration values as readonly fields.

---

## 5. TypeScript Access Modifiers (`public`, `private`, `protected`)

### Definitions

**Core Definition**
Access modifiers in TypeScript control the visibility of class members. `public` (default) allows access from anywhere. `private` restricts access to the declaring class. `protected` restricts access to the declaring class and its subclasses. These modifiers are compile-time-only constraints.

**Technical Definition**
TypeScript access modifiers are type-level annotations that control member visibility during type checking. `public` members are accessible from any code. `private` members are accessible only within the declaring class (not subclasses). `protected` members are accessible within the declaring class and its subclasses. These modifiers are enforced at compile time and erased at runtime—`private` and `protected` members are still accessible at runtime via bracket notation or type assertions. Classes with `private` or `protected` members are compared nominally (structural compatibility requires the same declaration), while classes with only `public` members are compared structurally.

**Beginner-Friendly Explanation**
Access modifiers control who can see and use class members. `public` means everyone can access it. `private` means only the class itself can access it—not even subclasses. `protected` means the class and its subclasses can access it. These are compile-time checks: TypeScript enforces them when you write code, but at runtime, the members are still there. This is different from JavaScript's `#private` fields, which are truly private at runtime. Access modifiers are primarily for documentation and compile-time safety.

### Purposes

- To encapsulate internal state and prevent external access.
- To document the intended visibility of class members.
- To enforce encapsulation at compile time.
- To enable nominal-like typing for classes with private members.
- To support inheritance hierarchies with protected members.

### Syntax Rules and Structure

**General Syntax: Access Modifiers**

```typescript
class ClassName {
  public publicField: Type;
  private privateField: Type;
  protected protectedField: Type;

  public publicMethod(): void { }
  private privateMethod(): void { }
  protected protectedMethod(): void { }
}
```

**Component Breakdown**
- `public`: Accessible from anywhere (default).
- `private`: Accessible only within the declaring class.
- `protected`: Accessible within the declaring class and subclasses.

**General Syntax: Modifier on Constructor Parameters**

```typescript
class ClassName {
  constructor(
    public publicParam: Type1,
    private privateParam: Type2,
    protected protectedParam: Type3
  ) {}
}
```

**Component Breakdown**
- Modifiers on constructor parameters create parameter properties.

**Syntax Rules**

- `public` is the default and can be omitted.
- `private` members are accessible only within the declaring class.
- `protected` members are accessible within the declaring class and subclasses.
- Modifiers apply to fields, methods, accessors, and constructor parameters.
- Classes with `private` or `protected` members are compared nominally.
- Access modifiers are erased at runtime.
- The `private` modifier does not prevent runtime access (use `#private` for that).

**Constraints and Limitations**

- `private` and `protected` are compile-time only; they provide no runtime protection.
- `private` members cannot be accessed in subclasses (unlike `protected`).
- Nominal comparison for classes with private members can cause unexpected incompatibility.
- Access modifiers cannot be used on static members in some older TypeScript versions (they can in modern versions).
- Bracket notation can bypass `private` at runtime in JavaScript.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Access Modifiers in Action

```typescript
// Step 1: Define a class with all access modifiers.
class BankAccount {
  public accountNumber: string;       // Accessible everywhere
  private balance: number;            // Accessible only in BankAccount
  protected owner: string;            // Accessible in BankAccount and subclasses

  constructor(accountNumber: string, owner: string, initialBalance: number) {
    this.accountNumber = accountNumber;
    this.owner = owner;
    this.balance = initialBalance;
  }

  public deposit(amount: number): void {
    this.balance += amount;  // ✅ Private access within the class
    console.log(`Deposited ${amount}. New balance: ${this.balance}`);
  }

  public getBalance(): number {
    return this.balance;  // ✅ Private access within the class
  }

  private validateAmount(amount: number): boolean {
    return amount > 0;
  }
}

// Step 2: Create an instance.
const account = new BankAccount("ACC-001", "Alice", 1000);

// Step 3: Public members are accessible.
console.log(account.accountNumber);  // "ACC-001"
account.deposit(500);                // "Deposited 500. New balance: 1500"
console.log(account.getBalance());   // 1500

// Step 4: Private members are NOT accessible.
// console.log(account.balance);     // ❌ Error: Property 'balance' is private.
// account.validateAmount(100);      // ❌ Error: Property 'validateAmount' is private.

// Step 5: Protected members are accessible in subclasses.
class SavingsAccount extends BankAccount {
  private interestRate: number;

  constructor(accountNumber: string, owner: string, initialBalance: number, interestRate: number) {
    super(accountNumber, owner, initialBalance);
    this.interestRate = interestRate;
  }

  public addInterest(): void {
    // this.balance is NOT accessible (private)
    // this.owner IS accessible (protected)
    console.log(`Adding interest for ${this.owner}`);
    const interest = this.getBalance() * this.interestRate;
    this.deposit(interest);
  }
}

const savings = new SavingsAccount("SAV-001", "Bob", 5000, 0.05);
savings.addInterest();  // "Adding interest for Bob" then "Deposited 250..."
```

**Expected Output:**
```
ACC-001
Deposited 500. New balance: 1500
1500
Adding interest for Bob
Deposited 250. New balance: 5250
```

**Why This Output Occurs:** `accountNumber` (public) is accessible everywhere. `balance` (private) is only accessible within `BankAccount` (used in `deposit` and `getBalance`). `owner` (protected) is accessible in both `BankAccount` and `SavingsAccount`. The subclass accesses `this.owner` (protected) but not `this.balance` (private), using `getBalance()` instead.

### Real-World Cases

**Case 1: Encapsulation in Domain Models**
Domain entities use `private` for internal state (balance, status) and `public` for methods that expose safe operations.

**Case 2: Framework Base Classes**
Framework base classes use `protected` for members that subclasses should access but external code should not.

**Case 3: Nominal Typing**
Classes with `private` members are compared nominally, which is useful for creating distinct types that happen to have the same shape.

---

## 6. JavaScript Hard Private Fields (`#private`)

### Definitions

**Core Definition**
JavaScript hard private fields use the `#` prefix to create truly private class fields that are enforced at runtime, not just at compile time. Unlike TypeScript's `private` modifier, `#private` fields cannot be accessed outside the class under any circumstances—not even via bracket notation or type assertions.

**Technical Definition**
Hard private fields, introduced in ES2022 and supported by TypeScript 3.8+, use the syntax `#fieldName` for declarations and access. These fields are stored in a private slot that is not accessible via property access, bracket notation, `Object.keys`, or reflection APIs. The `#` prefix is part of the field name, and the fields must be declared in the class body. TypeScript enforces `#private` access at compile time and emits the `#` syntax to JavaScript, which enforces it at runtime. Subclasses cannot access parent class `#private` fields.

**Beginner-Friendly Explanation**
JavaScript's `#private` fields are truly private—no one outside the class can access them, not even with tricks. TypeScript's `private` keyword is only a compile-time check: at runtime, the field is still accessible. If you need real privacy (e.g., for security or to prevent accidental access), use `#private`. The `#` symbol before the field name marks it as hard private. Subclasses can't access parent `#private` fields either, making them completely encapsulated.

### Purposes

- To create truly private fields that are inaccessible at runtime.
- To prevent accidental access or modification of internal state.
- To provide runtime-enforced encapsulation (unlike TypeScript's `private`).
- To avoid naming collisions in inheritance hierarchies.
- To satisfy security requirements where runtime privacy is essential.

### Syntax Rules and Structure

**General Syntax: Hard Private Field**

```typescript
class ClassName {
  #privateField: Type;

  constructor(value: Type) {
    this.#privateField = value;
  }

  #privateMethod(): ReturnType {
    return this.#privateField;
  }
}
```

**Component Breakdown**
- `#privateField`: The `#` prefix marks the field as hard private.
- Access uses `this.#privateField` within the class.
- External access is a compile-time and runtime error.

**Syntax Rules**

- Hard private fields use the `#` prefix in both declaration and access.
- `#private` fields must be declared in the class body (not in the constructor).
- Access is only allowed within the declaring class.
- Subclasses cannot access parent `#private` fields.
- `#private` fields are not enumerable and do not appear in `Object.keys` or `for...in`.
- `#private` methods are supported (TypeScript 4.3+).
- `#private` fields cannot be combined with TypeScript access modifiers.

**Constraints and Limitations**

- `#private` fields cannot be accessed by subclasses (unlike `protected`).
- `#private` fields cannot be used with `readonly` modifier (though they are effectively read-only from outside).
- `#private` fields require ES2022+ target or downlevel support.
- Reflection APIs cannot access `#private` fields.
- `#private` fields cannot be used with parameter properties.
- Mixing `#private` and TypeScript `private` in the same class is allowed but can be confusing.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Hard Private Fields vs TypeScript `private`

```typescript
// Step 1: Define a class with both TypeScript private and hard private fields.
class SecureAccount {
  private tsPrivate: string = "typescript-private";
  #hardPrivate: string = "hard-private";

  getTsPrivate(): string {
    return this.tsPrivate;
  }

  getHardPrivate(): string {
    return this.#hardPrivate;  // ✅ Accessible within the class
  }
}

// Step 2: Create an instance.
const account = new SecureAccount();

// Step 3: TypeScript private is accessible via bracket notation at runtime.
console.log(account.getTsPrivate());  // "typescript-private"
console.log((account as any).tsPrivate);  // "typescript-private" — bypassed!

// Step 4: Hard private is NOT accessible even with bracket notation.
console.log(account.getHardPrivate());  // "hard-private"
// console.log((account as any)["#hardPrivate"]);  // undefined
// console.log((account as any).hardPrivate);      // undefined

// Step 5: Hard private fields don't appear in enumeration.
console.log(Object.keys(account));  // ["tsPrivate"] — only the TS private field
```

**Expected Output:**
```
typescript-private
typescript-private
hard-private
[ 'tsPrivate' ]
```

**Why This Output Occurs:** TypeScript's `private` modifier is compile-time only, so `(account as any).tsPrivate` bypasses it at runtime. The `#hardPrivate` field is truly private: it doesn't appear in `Object.keys`, and bracket notation cannot access it. The `#` prefix is part of the field name and is enforced by JavaScript's runtime.

#### Example 2: Hard Private Methods

```typescript
// Step 1: Define a class with hard private methods.
class Counter {
  #count = 0;

  #validateIncrement(amount: number): void {
    if (amount < 0) {
      throw new Error("Increment must be non-negative");
    }
  }

  increment(amount: number = 1): void {
    this.#validateIncrement(amount);  // ✅ Accessible within the class
    this.#count += amount;
  }

  get count(): number {
    return this.#count;
  }
}

// Step 2: Use the class.
const counter = new Counter();
counter.increment();
counter.increment(5);
console.log(counter.count);  // 6

// Step 3: Hard private methods are not accessible.
// counter.#validateIncrement(1);  // ❌ Error: Property '#validateIncrement' is not accessible.

// Step 4: Hard private fields cannot be accessed.
// console.log(counter.#count);  // ❌ Error: Property '#count' is not accessible.
```

**Expected Output:**
```
6
```

**Why This Output Occurs:** The `#validateIncrement` method and `#count` field are hard private, accessible only within the `Counter` class. The public `increment` method uses them internally, and the public `count` getter exposes the count safely.

### Real-World Cases

**Case 1: Security-Sensitive State**
Classes that handle sensitive data (authentication tokens, encryption keys) use `#private` to prevent runtime access.

**Case 2: Library Internal State**
Libraries that need true encapsulation of internal state use `#private` to prevent consumers from relying on implementation details.

**Case 3: Avoiding Inheritance Collisions**
When a class hierarchy might use the same field names, `#private` ensures that parent and child fields are truly separate.

---

## 7. Getters and Setters (`get` / `set` Accessors)

### Definitions

**Core Definition**
Getters and setters are special methods that provide controlled access to a property. A getter (`get`) is called when the property is read; a setter (`set`) is called when the property is assigned. Together, they allow computed properties, validation, and side effects during property access.

**Technical Definition**
TypeScript supports ES5+ getter and setter syntax in classes. A getter is declared with `get propertyName(): ReturnType { }` and is invoked on property read. A setter is declared with `set propertyName(value: ParamType) { }` and is invoked on property write. Getters and setters must have the same property name, and if both are present, the setter's parameter type must be assignable to the getter's return type (TypeScript 4.3+). Accessors can have access modifiers (`public`, `private`, `protected`) and `static`. Since TypeScript 4.3, getters and setters can have different access modifiers (e.g., public getter, private setter).

**Beginner-Friendly Explanation**
Getters and setters let you run code when someone reads or writes a property. Instead of a plain field, you can have a `get` method that computes a value on the fly, or a `set` method that validates input before storing it. For example, a `fullName` getter might combine `firstName` and `lastName`, and a `set` method might reject invalid emails. They look like properties from the outside—`user.fullName` calls the getter, and `user.email = "..."` calls the setter. This encapsulation lets you change internal representation without changing the public API.

### Purposes

- To provide computed properties derived from other fields.
- To validate or transform values before storing them.
- To trigger side effects when a property is accessed or modified.
- To encapsulate internal representation while exposing a property-like API.
- To implement lazy initialization of expensive values.

### Syntax Rules and Structure

**General Syntax: Getter and Setter**

```typescript
class ClassName {
  private _field: Type;

  get propertyName(): Type {
    return this._field;
  }

  set propertyName(value: Type) {
    this._field = value;
  }
}
```

**Component Breakdown**
- `get propertyName(): Type`: The getter, invoked on read.
- `set propertyName(value: Type)`: The setter, invoked on write.
- The backing field (`_field`) stores the actual value.

**General Syntax: Getter Only (Read-Only Property)**

```typescript
class ClassName {
  get computed(): ReturnType {
    return /* computed value */;
  }
}
```

**Component Breakdown**
- A getter without a setter creates a read-only property.

**General Syntax: Different Access Modifiers (TypeScript 4.3+)**

```typescript
class ClassName {
  get value(): Type { return this._value; }
  private set value(v: Type) { this._value = v; }
}
```

**Component Breakdown**
- Public getter, private setter.

**Syntax Rules**

- Getters use `get name(): Type { }`.
- Setters use `set name(value: Type) { }`.
- A getter without a setter creates a read-only property.
- A setter without a getter creates a write-only property (rare).
- The getter's return type and the setter's parameter type must be compatible (TypeScript 4.3+).
- Accessors can be `static`, `public`, `private`, or `protected`.
- Getters and setters can have different access modifiers (TypeScript 4.3+).
- Accessors cannot have the same name as a field in the same class.

**Constraints and Limitations**

- Before TypeScript 4.3, getters and setters had to have the same access modifier.
- Accessors add runtime overhead compared to plain fields.
- Accessors cannot be used with parameter properties.
- A getter without a setter means the property is read-only (assignment is a compile error).
- Accessors and fields cannot share the same name in the same class.
- `super` accessors work but require care with inheritance.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Getter and Setter

```typescript
// Step 1: Define a class with a getter and setter.
class Temperature {
  private _celsius: number = 0;

  get celsius(): number {
    return this._celsius;
  }

  set celsius(value: number) {
    if (value < -273.15) {
      throw new Error("Temperature below absolute zero is not possible");
    }
    this._celsius = value;
  }

  get fahrenheit(): number {
    return (this._celsius * 9) / 5 + 32;
  }

  set fahrenheit(value: number) {
    this.celsius = ((value - 32) * 5) / 9;  // Uses the celsius setter
  }
}

// Step 2: Use the accessors.
const temp = new Temperature();
temp.celsius = 25;
console.log(`Celsius: ${temp.celsius}`);       // 25
console.log(`Fahrenheit: ${temp.fahrenheit}`); // 77

temp.fahrenheit = 100;
console.log(`Celsius: ${temp.celsius}`);       // 37.777...
console.log(`Fahrenheit: ${temp.fahrenheit}`); // 100

// Step 3: Setter validation.
try {
  temp.celsius = -300;
} catch (e) {
  console.log(`Error: ${(e as Error).message}`);
}
```

**Expected Output:**
```
Celsius: 25
Fahrenheit: 77
Celsius: 37.77777777777778
Fahrenheit: 100
Error: Temperature below absolute zero is not possible
```

**Why This Output Occurs:** The `celsius` getter returns the backing field, and the setter validates before storing. The `fahrenheit` getter computes from `celsius`, and the setter converts and delegates to the `celsius` setter. Validation in the setter prevents invalid temperatures.

#### Example 2: Different Access Modifiers for Getter and Setter

```typescript
// Step 1: Define a class with a public getter and private setter.
class User {
  private _email: string = "";

  get email(): string {
    return this._email;
  }

  private set email(value: string) {
    if (!value.includes("@")) {
      throw new Error("Invalid email address");
    }
    this._email = value;
  }

  constructor(email: string) {
    this.email = email;  // ✅ Allowed inside the class
  }

  updateEmail(newEmail: string): void {
    this.email = newEmail;  // ✅ Allowed inside the class
  }
}

// Step 2: Use the class.
const user = new User("alice@example.com");
console.log(user.email);  // "alice@example.com"

user.updateEmail("alice.smith@example.com");
console.log(user.email);  // "alice.smith@example.com"

// Step 3: The setter is private — external assignment fails.
// user.email = "bob@example.com";  // ❌ Error: Property 'email' has no setter.
```

**Expected Output:**
```
alice@example.com
alice.smith@example.com
```

**Why This Output Occurs:** The getter is public, so `user.email` is readable externally. The setter is private, so external code cannot assign `user.email = ...`. The constructor and `updateEmail` method can use the setter internally.

### Real-World Cases

**Case 1: Validated Properties**
Classes that store user input (email, phone, age) use setters for validation, ensuring only valid values are stored.

**Case 2: Computed Properties**
Classes with derived properties (fullName from firstName + lastName, area from width + height) use getters to compute values on demand.

**Case 3: Lazy Initialization**
Expensive resources (database connections, large computations) are lazily initialized in getters, avoiding the cost until first access.

---

## 8. `readonly` Properties in Classes

### Definitions

**Core Definition**
The `readonly` modifier in TypeScript classes makes a property immutable after initialization. Readonly properties can only be assigned during declaration or in the constructor (and in the constructor of the declaring class only—not subclasses). This is a compile-time-only constraint.

**Technical Definition**
Class properties marked `readonly` can only be assigned at the point of declaration or within the constructor of the declaring class. Assignments in subclass constructors or instance methods are compile errors. The `readonly` modifier can be combined with access modifiers (`public readonly`, `private readonly`, `protected readonly`) and parameter properties (`constructor(readonly name: string)`). Readonly is shallow: if the property holds an object, the object's properties can still be mutated. `readonly` is erased at runtime and does not prevent property modification via type assertions or bracket notation.

**Beginner-Friendly Explanation**
A readonly property is one that can't be changed after the object is created. You can set it when you declare it or in the constructor, but after that, it's locked. This is useful for properties that should never change—like IDs, creation timestamps, or configuration values. For example, `readonly id: number` means once the object is created with an ID, no one can change it. But be careful: readonly only applies to the property itself. If the property holds an object or array, you can still change the contents of that object or array.

### Purposes

- To prevent accidental modification of properties that should remain constant.
- To document the intended immutability of class properties.
- To enable compile-time enforcement of immutability for IDs, timestamps, and configuration.
- To support immutable value objects and data classes.
- To combine with parameter properties for concise immutable classes.

### Syntax Rules and Structure

**General Syntax: Readonly Property**

```typescript
class ClassName {
  readonly propertyName: Type = initialValue;

  constructor(value: Type) {
    this.readonlyProperty = value;  // ✅ Allowed in the declaring class constructor
  }
}
```

**Component Breakdown**
- `readonly`: The modifier preventing reassignment.
- Assignment allowed at declaration and in the declaring class's constructor.

**General Syntax: Readonly Parameter Property**

```typescript
class ClassName {
  constructor(readonly propertyName: Type) {}
}
```

**Component Breakdown**
- `readonly` on a constructor parameter creates a readonly field.

**General Syntax: Readonly with Access Modifier**

```typescript
class ClassName {
  public readonly publicReadonly: Type;
  private readonly privateReadonly: Type;
  protected readonly protectedReadonly: Type;
}
```

**Component Breakdown**
- Modifiers can be combined: `public readonly`, `private readonly`, `protected readonly`.

**Syntax Rules**

- `readonly` properties can only be assigned at declaration or in the declaring class's constructor.
- Subclass constructors cannot assign parent readonly properties.
- `readonly` can be combined with access modifiers.
- `readonly` parameter properties create readonly fields.
- `readonly` is shallow: nested object properties remain mutable.
- `readonly` is erased at runtime and provides no runtime protection.
- `as const` cannot be used on class properties (only object literals and arrays).

**Constraints and Limitations**

- `readonly` is shallow: nested objects and arrays remain mutable.
- `readonly` is erased at runtime and can be bypassed with type assertions.
- Subclasses cannot assign parent readonly properties.
- `readonly` cannot be used with `#private` fields (though `#private` fields are effectively read-only from outside).
- Readonly properties cannot be conditionally assigned in the constructor (only once).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Readonly Properties in Classes

```typescript
// Step 1: Define a class with readonly properties.
class User {
  readonly id: number;
  readonly createdAt: Date;
  name: string;  // Mutable

  constructor(id: number, name: string) {
    this.id = id;                        // ✅ Allowed in constructor
    this.createdAt = new Date();         // ✅ Allowed in constructor
    this.name = name;
  }

  updateName(newName: string): void {
    this.name = newName;  // ✅ Mutable field
    // this.id = 999;     // ❌ Error: Cannot assign to 'id' because it is a read-only property.
  }
}

// Step 2: Create an instance.
const alice = new User(1, "Alice");
console.log(`ID: ${alice.id}, Name: ${alice.name}`);

// Step 3: Readonly properties cannot be reassigned.
// alice.id = 2;  // ❌ Error: Cannot assign to 'id'.

// Step 4: Mutable properties can be reassigned.
alice.name = "Alice Smith";  // ✅ Allowed
console.log(`Updated name: ${alice.name}`);

// Step 5: Shallow readonly — the Date object is still mutable.
alice.createdAt.setFullYear(2025);  // ✅ Allowed — mutating the Date object
console.log(`Modified createdAt: ${alice.createdAt.getFullYear()}`);
```

**Expected Output:**
```
ID: 1, Name: Alice
Updated name: Alice Smith
Modified createdAt: 2025
```

**Why This Output Occurs:** The `id` and `createdAt` properties are readonly, so they can only be assigned in the constructor. The `name` property is mutable and can be reassigned. The `createdAt` Date object is mutable even though the property is readonly (shallow readonly).

#### Example 2: Readonly Parameter Properties and Inheritance

```typescript
// Step 1: Define a base class with readonly parameter properties.
class Entity {
  constructor(
    public readonly id: number,
    public readonly createdAt: Date = new Date()
  ) {}
}

// Step 2: Define a subclass.
class User extends Entity {
  constructor(
    id: number,
    public name: string,
    public email: string
  ) {
    super(id);  // ✅ Passes id to Entity constructor
    // this.id = id;  // ❌ Error: Cannot assign to 'id' — it's readonly and belongs to Entity.
  }
}

// Step 3: Create instances.
const user = new User(1, "Alice", "alice@example.com");
console.log(`User ${user.id}: ${user.name} (${user.email})`);
console.log(`Created: ${user.createdAt.toISOString()}`);

// Step 4: Readonly properties cannot be modified.
// user.id = 2;  // ❌ Error: Cannot assign to 'id'.

// Step 5: Mutable properties can be modified.
user.name = "Alice Smith";  // ✅ Allowed
console.log(`Updated: ${user.name}`);
```

**Expected Output:**
```
User 1: Alice (alice@example.com)
Created: 2024-...
Updated: Alice Smith
```

**Why This Output Occurs:** The `Entity` class declares `id` and `createdAt` as readonly parameter properties. The `User` subclass passes `id` to the parent constructor via `super(id)`. The subclass cannot assign `this.id` because readonly properties can only be assigned in the declaring class's constructor.

### Real-World Cases

**Case 1: Entity IDs and Timestamps**
Domain entities use readonly for `id` and `createdAt` fields, ensuring these values never change after creation.

**Case 2: Configuration Objects**
Configuration classes use readonly for values that should not change at runtime, documenting and enforcing immutability.

**Case 3: Value Objects**
Value objects (Money, Coordinates, DateRange) use readonly for all fields, making them truly immutable value types.

---

## 9. Static Members (Properties, Blocks, and Methods)

### Definitions

**Core Definition**
Static members belong to the class itself, not to instances. Static properties, methods, and initialization blocks are accessed via the class name (`ClassName.staticMember`) rather than through instances. Static members are shared across all instances and are commonly used for utility functions, constants, and class-level state.

**Technical Definition**
Static members are declared with the `static` keyword. Static properties are stored on the constructor function. Static methods are called on the class itself and have `this` typed as the class constructor (not an instance). Static initialization blocks (`static { }`), introduced in TypeScript 4.4 / ES2022, run once when the class is defined, enabling complex static initialization with access to private static fields. Static members can have access modifiers (`public`, `private`, `protected`) and can be `readonly`. Static members are inherited by subclasses but accessed via the subclass name. Static members cannot access instance members directly and vice versa.

**Beginner-Friendly Explanation**
Static members belong to the class, not to individual objects. If you have a `MathUtils` class with a static `add` method, you call it as `MathUtils.add(1, 2)`—you don't need to create an instance. Static properties are shared across all instances, like a counter that tracks how many objects have been created. Static blocks are a way to run initialization code for static properties. Static members are useful for utility functions, constants, factory methods, and class-level state.

### Purposes

- To define utility functions and constants that don't depend on instance state.
- To track class-level state (e.g., instance counts, registries).
- To implement factory methods that create instances in controlled ways.
- To perform complex static initialization with static blocks.
- To share data across all instances of a class.

### Syntax Rules and Structure

**General Syntax: Static Property**

```typescript
class ClassName {
  static staticProperty: Type = initialValue;
}
```

**Component Breakdown**
- `static`: The keyword placing the property on the class itself.
- Access: `ClassName.staticProperty`.

**General Syntax: Static Method**

```typescript
class ClassName {
  static staticMethod(param: ParamType): ReturnType {
    return /* ... */;
  }
}
```

**Component Breakdown**
- `static`: The keyword placing the method on the class itself.
- Access: `ClassName.staticMethod(args)`.

**General Syntax: Static Block**

```typescript
class ClassName {
  static staticProperty: Type;

  static {
    // Initialization logic
    ClassName.staticProperty = computeValue();
  }
}
```

**Component Breakdown**
- `static { }`: A static initialization block that runs once.
- Multiple static blocks run in declaration order.

**General Syntax: Static with Access Modifiers**

```typescript
class ClassName {
  private static privateStatic: Type;
  protected static protectedStatic: Type;
  public static publicStatic: Type;
  static readonly readonlyStatic: Type = value;
}
```

**Component Breakdown**
- Static members can have access modifiers and `readonly`.

**Syntax Rules**

- Static members are declared with the `static` keyword.
- Static members are accessed via the class name, not instances.
- Static methods have `this` typed as the class constructor (not an instance).
- Static blocks (`static { }`) run once when the class is defined.
- Static members can have `public`, `private`, `protected`, and `readonly` modifiers.
- Static members are inherited by subclasses (accessible via subclass name).
- Static members cannot directly access instance members.
- Instance members cannot directly access static members without using the class name.

**Constraints and Limitations**

- Static members cannot access instance members directly (no `this` instance context).
- Instance members must use `ClassName.staticMember` to access static members.
- Static methods cannot be polymorphic with the `this` type (static `this` is the constructor).
- Static blocks cannot access instance fields.
- Static members are not available on instances (e.g., `instance.staticMethod` is undefined).
- Private static members are accessible within the class body only.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Static Properties and Methods

```typescript
// Step 1: Define a class with static members.
class Counter {
  static count: number = 0;              // Static property
  static readonly MAX: number = 100;     // Static readonly
  private static instances: number = 0;  // Private static

  id: number;

  constructor() {
    Counter.instances += 1;
    this.id = Counter.instances;
    Counter.count += 1;
  }

  static getInstanceCount(): number {
    return Counter.instances;
  }

  static reset(): void {
    Counter.count = 0;
    // Counter.MAX = 200;  // ❌ Error: Cannot assign to 'MAX' because it is read-only.
  }

  static create(): Counter {
    return new Counter();  // Factory method
  }
}

// Step 2: Use static members.
const c1 = new Counter();
const c2 = new Counter();
const c3 = Counter.create();  // Factory method

console.log(`Instances: ${Counter.getInstanceCount()}`);  // 3
console.log(`Count: ${Counter.count}`);                   // 3
console.log(`MAX: ${Counter.MAX}`);                       // 100

// Step 3: Static members are NOT on instances.
// console.log(c1.count);  // ❌ Error: Property 'count' does not exist on type 'Counter'.

// Step 4: Reset static state.
Counter.reset();
console.log(`After reset — Count: ${Counter.count}`);  // 0
```

**Expected Output:**
```
Instances: 3
Count: 3
MAX: 100
After reset — Count: 0
```

**Why This Output Occurs:** The static `instances` and `count` properties are shared across all instances. Each constructor increments both. The `getInstanceCount` and `reset` methods are static, accessed via `Counter`. The `MAX` static readonly cannot be reassigned.

#### Example 2: Static Initialization Blocks

```typescript
// Step 1: Define a class with static initialization blocks.
class Config {
  static readonly apiUrl: string;
  static readonly timeout: number;
  static readonly features: string[];

  static {
    // Complex initialization logic
    const env = process.env.NODE_ENV ?? "development";
    Config.apiUrl = env === "production"
      ? "https://api.example.com"
      : "http://localhost:3000";
    Config.timeout = env === "production" ? 10000 : 5000;
    Config.features = env === "production"
      ? ["auth", "logging", "metrics"]
      : ["auth", "logging"];
    console.log(`Config initialized for ${env}`);
  }

  // Multiple static blocks run in order.
  static {
    console.log(`Features: ${Config.features.join(", ")}`);
  }
}

// Step 2: Static blocks run when the class is defined.
// Output (in development):
// "Config initialized for development"
// "Features: auth, logging"

// Step 3: Access static properties.
console.log(Config.apiUrl);   // "http://localhost:3000"
console.log(Config.timeout);  // 5000
```

**Expected Output:**
```
Config initialized for development
Features: auth, logging
http://localhost:3000
5000
```

**Why This Output Occurs:** The static blocks run once when the `Config` class is defined. The first block reads `process.env.NODE_ENV` and initializes `apiUrl`, `timeout`, and `features`. The second block logs the features. Both blocks run before any instance is created.

### Real-World Cases

**Case 1: Utility Classes**
Utility classes (MathUtils, StringUtils) use static methods for pure functions that don't depend on instance state.

**Case 2: Singleton Pattern**
Singleton classes use a private static instance property and a static `getInstance` method to ensure only one instance exists.

**Case 3: Factory Methods**
Classes use static factory methods (`fromJSON`, `create`, `of`) to provide named constructors and controlled instantiation.

**Case 4: Class-Level Registries**
Classes maintain static registries (e.g., of all instances, of subclasses) for plugin systems and dependency injection.

---

## References

- TypeScript Handbook: Classes — https://www.typescriptlang.org/docs/handbook/2/classes.html
- TypeScript Handbook: Class Members (Access Modifiers, Readonly, Static) — https://www.typescriptlang.org/docs/handbook/2/classes.html#class-members
- TypeScript 4.3 Release Notes (Separate Write Types on Properties) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-3.html
- TypeScript 4.4 Release Notes (Static Blocks) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html
- TypeScript 3.8 Release Notes (ECMAScript Private Fields) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html
- TypeScript `useDefineForClassFields` Documentation — https://www.typescriptlang.org/tsconfig#useDefineForClassFields
- MDN: Classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- MDN: Private Class Fields — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields
- MDN: Static Initialization Blocks — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Static_initialization_blocks
- MDN: getter — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get
- MDN: setter — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set
- TypeScript Playground: Classes — https://www.typescriptlang.org/play/typescript/classes.ts.html
- Effective TypeScript: Item 28 — Prefer Classes to Namespaces
- Total TypeScript: Classes — https://www.totaltypescript.com/tutorials/beginners-typescript/12-classes
- TypeScript ESLint: explicit-member-accessibility — https://typescript-eslint.io/rules/explicit-member-accessibility/