# Generic Patterns in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Generic patterns** are reusable software design structures in which **type parameters** are used to abstract over the concrete types involved, allowing a single implementation to operate across multiple types while preserving static type relationships. In TypeScript, generic patterns leverage parameterized type aliases, interfaces, classes, and functions to express creational and structural designs — such as Factory, Repository, and Builder — in a **type-safe and inference-friendly** manner.

**Technical Definition.** A generic pattern is a design pattern whose participants are parameterized by one or more type parameters. The type parameters are declared on the pattern’s core abstraction (a function, class, or interface) and are propagated through its operations so that the compiler can verify that inputs, outputs, and internal state remain consistently typed. TypeScript’s structural type system, inference engine, and type-level operators (conditional types, mapped types, `infer`) make it possible to encode pattern invariants — such as “the builder cannot be built until required fields are set” — directly in the type system.

**Beginner-Friendly Explanation.** A generic pattern is **a reusable recipe that works with many types without losing track of which type you are using**. Instead of writing a separate factory for `User`, another for `Post`, and another for `Comment`, you write **one factory** that works for all of them and still knows that a `User` factory produces a `User`. The “generic” part is the placeholder (`<T>`) that gets filled in with the actual type when you use the pattern.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Type parameterization** | The pattern’s core abstraction is parameterized by one or more type parameters, enabling reuse across types. |
| **Type preservation** | Generic patterns preserve the relationship between input and output types, unlike `any`-based patterns that lose type information. |
| **Constraint-driven safety** | Type parameters are constrained (`extends`) to ensure only compatible types are accepted. |
| **Inference-friendliness** | Well-designed generic patterns let callers omit type arguments, relying on inference from values. |
| **Accumulated type state** | Advanced patterns (Builder, type-safe maps) accumulate type information across method calls using intersection types and generic capture. |
| **Erasure-compatible** | Generic patterns compile to plain JavaScript; type parameters and constraints are erased at runtime. |

---

### Prerequisites

- **Generics fundamentals**: declaring `<T>`, constraints (`<T extends U>`), and type arguments.
- **Type inference**: how TypeScript infers type arguments from function arguments and contextual positions.
- **Object-oriented and functional TypeScript**: classes, interfaces, type aliases, and functions.
- **Utility types**: `Partial<T>`, `Omit<T, K>`, `Record<K, V>`, and `InstanceType<T>`.

---

### Related Programming Areas

- **Design patterns** (Gang of Four): Factory, Builder, Repository, Strategy, Abstract Factory.
- **Domain-Driven Design (DDD)** : Repository pattern as a domain-layer abstraction.
- **Dependency injection**: generic factories for composition.
- **Type-level programming**: conditional types, mapped types, and type-state tracking.
- **Functional programming**: generic higher-order functions and combinators.

---

### Core Concepts / Features

The following five sections cover generic patterns as a discipline, followed by the three foundational creational/structural patterns and best practices.

---

## 1. Generic Patterns (General)

### Definitions

**Core Definition.** A **generic pattern** is a design pattern in which type parameters are used to abstract over the concrete types involved, so that the pattern’s structure and invariants are expressed and enforced at the type level rather than duplicated for each concrete type.

**Technical Definition.** Generic patterns exploit TypeScript’s **parametric polymorphism**: a single declaration can be instantiated with different type arguments, and the compiler checks that operations within the pattern are valid for all permitted instantiations. Constraints (`extends`) restrict the set of permissible type arguments. Inference allows type arguments to be derived from values, and type-level operators allow the pattern’s output type to be computed from its input types.

**Beginner-Friendly Explanation.** A generic pattern is **a template for solving a recurring design problem that works with any type**. It is “generic” because it does not hard-code `User` or `Order`; it uses a placeholder type that you fill in. The pattern still guarantees that the types line up correctly.

### Purposes

- **To eliminate duplicated pattern implementations** across different entity types.
- **To preserve static type relationships** between inputs, outputs, and internal state across the pattern’s operations.
- **To enforce pattern invariants at compile time**, such as “required fields must be set before building”.
- **To enable inference-friendly APIs** so callers rarely need to spell out type arguments.
- **To align implementation structure with domain abstractions** without sacrificing type safety.

### Syntax Rules and Structure

Generic patterns use the standard generic syntax of TypeScript. The core syntactic elements are:

1. **Type parameter lists**: `<T>`, `<T, U>`, `<T extends Constraint>`.
2. **Parameterized abstractions**: generic functions, classes, interfaces, and type aliases.
3. **Type-level operators**: `keyof`, `typeof`, conditional types (`T extends U ? X : Y`), mapped types, and `infer`.

**Component breakdown (generic class declaration):**

```
class Repository<T extends { id: string }> {
  // ...
}
```

- `class` — declares a class.
- `Repository` — the class name.
- `<T extends { id: string }>` — the **type parameter list**: `T` is the parameter, and `{ id: string }` is the **constraint**.
- The constraint ensures that only types with an `id: string` property can be used.

**Syntax rules.**

- Type parameters must be declared before they are used.
- Constraints limit the set of valid type arguments and unlock constraint-specific operations inside the pattern.
- Generic type parameters can be inferred from constructor arguments, function arguments, and return types.
- Default type parameters (`<T = DefaultType>`) may be used to provide fallback types.

**Constraints and limitations.**

- Over-parameterization can make APIs harder to use; use the **fewest type parameters necessary**.
- Type parameters that are not used in the pattern’s signature are a code smell and should be removed.
- Generic patterns are erased at runtime; they cannot inspect type arguments dynamically.

### Annotated Complete Step by Step Code Examples

#### Example 1: A generic identity pattern

```typescript
// example_generic_identity.ts
// Step 1: Declare a generic function with a type parameter T.
function identity<T>(value: T): T {
  return value; // T is used for both parameter and return type
}

// Step 2: Call with explicit type argument.
const a = identity<string>("hello");

// Step 3: Call with inference — T is inferred as number.
const b = identity(42);

console.log(a); // Expected: hello
console.log(b); // Expected: 42
```

**Expected Output:**
```
hello
42
```

**Why this output occurs.** `identity` preserves the input type. `identity<string>` fixes `T` to `string`; `identity(42)` infers `T` as `number` from the argument. The generic pattern ensures that the return type is always the same as the argument type, which `any` would not guarantee.

---

#### Example 2: A generic pair pattern with constraints

```typescript
// example_generic_pair.ts
// Step 1: Define a constrained generic interface.
interface Entity {
  id: string;
}

interface Pair<A extends Entity, B extends Entity> {
  first: A;
  second: B;
}

// Step 2: Use with concrete entity types.
const user = { id: "u1", name: "Ada" };
const post = { id: "p1", title: "Hello" };

const pair: Pair<typeof user, typeof post> = { first: user, second: post };

console.log(pair.first.name); // Expected: Ada
console.log(pair.second.title); // Expected: Hello
```

**Expected Output:**
```
Ada
Hello
```

**Why this output occurs.** The constraint `A extends Entity` and `B extends Entity` ensures that both type arguments have an `id: string` property. Inside `Pair`, the specific properties (`name`, `title`) remain available because `A` and `B` are preserved, not widened to `Entity`.

---

### Real-World Cases

- **Generic collections**: `Map<K, V>`, `Set<T>`, and custom `List<T>` abstractions.
- **Generic result types**: `Result<T, E>` for success/failure modeling.
- **Generic event emitters**: `EventEmitter<Events>` where `Events` maps event names to payload types.
- **Generic API response wrappers**: `ApiResponse<T>` preserving the payload type through HTTP layers.

---

## 2. Factory Pattern (Generic)

### Definitions

**Core Definition.** A **generic factory pattern** is a creational design pattern in which a function or method is parameterized by the type of object it creates, allowing a single factory implementation to produce instances of multiple types while preserving the concrete return type for each call.

**Technical Definition.** A generic factory declares a type parameter `T` (often constrained) and receives a **constructor reference** (`new (...args: any[]) => T`) or a **discriminated key** that determines the concrete class. The factory’s return type is `T` or a computed type based on the key, so the compiler knows exactly which type was produced. Constructor constraints use the `new () => T` syntax; parameterized constructors use `new (...args: Args) => T`.

**Beginner-Friendly Explanation.** A generic factory is **a single “object-making machine” that can make different kinds of objects and still tells you exactly what it made**. Instead of writing `createUser()`, `createPost()`, and `createComment()` separately, you write one `create(Constructor, ...args)` function that works for all of them. TypeScript tracks which class you passed in and gives you back the correct instance type.

### Purposes

- **To centralize object creation logic** while supporting multiple concrete types.
- **To preserve the concrete return type** so callers receive a fully typed instance, not a base type or `any`.
- **To enforce constructor compatibility** through constraints (`new () => T`), preventing the factory from being called with non-constructable values.
- **To enable dependency injection and registry-based creation** with compile-time safety.

### Syntax Rules and Structure

**General syntax (constructor-constrained factory):**

```
function create<T>(ctor: new (...args: any[]) => T): T {
  return new ctor();
}
```

**Component breakdown:**

- `function create` — the factory function.
- `<T>` — the type parameter representing the created instance type.
- `ctor: new (...args: any[]) => T` — a **constructor type**: `new` indicates constructability, `(...args: any[])` accepts constructor arguments, and `=> T` specifies the produced type.
- `return new ctor()` — instantiates the class and returns `T`.

**Registry-based factory syntax:**

```
function factory<K extends keyof typeof registry>(
  key: K
): InstanceType<(typeof registry)[K]> {
  const Ctor = registry[key];
  return new Ctor() as InstanceType<(typeof registry)[K]>;
}
```

- `K extends keyof typeof registry` — constrains the key to valid registry keys.
- `InstanceType<(typeof registry)[K]>` — computes the instance type of the selected constructor.
- This pattern preserves the specific return type for each key.

**Syntax rules.**

- Constructor constraints use `new () => T` for parameterless constructors and `new (...args: Args) => T` for parameterized constructors.
- The type parameter should appear in the return type; otherwise the generic is unused.
- For registry-based factories, use `keyof typeof` and indexed access types to preserve key-specific return types.
- Inference can often determine `T` from the constructor argument, eliminating explicit type arguments.

**Constraints and limitations.**

- TypeScript erases generic type information; a factory that relies on `new T()` without a constructor reference will fail at runtime.
- Constructor parameter types may require `ConstructorParameters<T>` to avoid `any`.
- Registry-based factories require the registry object to be `as const` or explicitly typed to preserve literal keys.

### Annotated Complete Step by Step Code Examples

#### Example 1: Constructor-constrained factory

```typescript
// example_factory_ctor.ts
// Step 1: Define a base interface.
interface Shape {
  area(): number;
}

// Step 2: Define two concrete classes.
class Circle implements Shape {
  constructor(private radius: number) {}
  area() {
    return Math.PI * this.radius ** 2;
  }
}

class Square implements Shape {
  constructor(private side: number) {}
  area() {
    return this.side ** 2;
  }
}

// Step 3: Generic factory constrained to constructors producing Shape.
function createShape<T extends Shape>(
  ctor: new (...args: any[]) => T,
  ...args: any[]
): T {
  return new ctor(...args);
}

// Step 4: Create instances with full type preservation.
const circle = createShape(Circle, 2);
const square = createShape(Square, 3);

console.log(circle.area().toFixed(2)); // Expected: 12.57
console.log(square.area());            // Expected: 9
```

**Expected Output:**
```
12.57
9
```

**Why this output occurs.** The factory is constrained to constructors that produce `Shape`. The type parameter `T` is inferred from the constructor argument: `Circle` yields `T = Circle`, and `Square` yields `T = Square`. The return types are therefore `Circle` and `Square`, not the widened `Shape`. This preserves access to class-specific members while enforcing the `Shape` contract.

---

#### Example 2: Registry-based factory with key-specific return types

```typescript
// example_factory_registry.ts
// Step 1: Define classes.
class Dog {
  speak() {
    return "Woof";
  }
}
class Cat {
  speak() {
    return "Meow";
  }
}

// Step 2: Create a registry with literal keys.
const registry = { dog: Dog, cat: Cat } as const;

// Step 3: Generic factory constrained to registry keys.
function createAnimal<K extends keyof typeof registry>(
  key: K
): InstanceType<(typeof registry)[K]> {
  const Ctor = registry[key];
  return new Ctor() as InstanceType<(typeof registry)[K]>;
}

// Step 4: Create animals with key-specific types.
const dog = createAnimal("dog");
const cat = createAnimal("cat");

console.log(dog.speak()); // Expected: Woof
console.log(cat.speak()); // Expected: Meow
```

**Expected Output:**
```
Woof
Meow
```

**Why this output occurs.** `K` is inferred as the literal `"dog"` or `"cat"`. The return type is computed via `InstanceType<(typeof registry)[K]>`, so `createAnimal("dog")` returns `Dog` and `createAnimal("cat")` returns `Cat`. This is a type-safe registry factory that preserves key-specific types without overloads.

---

### Real-World Cases

- **Dependency injection containers**: factories resolve and instantiate services by token, preserving the service type.
- **Test data builders**: generic factories create fixtures for multiple entity types with correct typing.
- **Plugin/strategy registries**: a registry maps strategy names to constructors, and the factory returns the correct strategy type.
- **ORM model factories**: generic factories produce model instances from database rows with type preservation.

---

## 3. Repository Pattern (Generic)

### Definitions

**Core Definition.** A **generic repository pattern** is a structural/data-access pattern in which a single repository abstraction is parameterized by the entity type it manages, providing type-safe CRUD and query operations for multiple entity types without duplicating data-access code.

**Technical Definition.** A generic repository defines an interface `IRepository<T>` with operations such as `findById`, `find`, `create`, `update`, and `delete`, all typed in terms of `T`. A `BaseRepository<T>` implements these operations against a data store, and concrete repositories extend the base with entity-specific queries. The type parameter `T` is constrained (commonly `T extends { id: string }` or `T extends Document`) to ensure required identity fields.

**Beginner-Friendly Explanation.** A generic repository is **one data-access class that works for every entity type**. Instead of writing `UserRepository`, `PostRepository`, and `CommentRepository` from scratch, you write one `Repository<T>` that knows how to find, create, update, and delete any `T`. Then you create thin concrete repositories only for entity-specific queries. TypeScript ensures that `UserRepository` returns `User` objects, not `any`.

### Purposes

- **To eliminate duplicated CRUD logic** across entity types.
- **To decouple business logic from data-access implementation**, improving testability.
- **To preserve entity types through the data-access layer**, so callers receive fully typed domain objects.
- **To enforce identity constraints** (`T extends { id: string }`) so operations like `findById` are always valid.
- **To provide a consistent extension point** for entity-specific query methods.

### Syntax Rules and Structure

**General syntax (generic repository interface):**

```
interface IRepository<T> {
  findById(id: string): Promise<T | null>;
  find(filter: Partial<T>): Promise<T[]>;
  create(data: Partial<T>): Promise<T>;
  updateById(id: string, update: Partial<T>): Promise<T | null>;
  deleteById(id: string): Promise<boolean>;
}
```

**Component breakdown:**

- `interface IRepository<T>` — the generic interface parameterized by entity type `T`.
- `findById(id: string): Promise<T | null>` — returns the entity or `null`.
- `find(filter: Partial<T>): Promise<T[]>` — accepts a partial entity as a filter.
- `create(data: Partial<T>): Promise<T>` — accepts creation data and returns the created entity.
- `Partial<T>` — allows partial specifications for filters and updates.

**Generic base class syntax:**

```
class BaseRepository<T extends { id: string }> implements IRepository<T> {
  constructor(protected model: Model<T>) {}
  // implementations...
}
```

- `T extends { id: string }` — constrains entities to have an identity field.
- `protected model: Model<T>` — the underlying data-store model, typed for `T`.

**Syntax rules.**

- Constrain `T` when the repository relies on identity or other structural properties (`T extends { id: string }`).
- Use `Partial<T>` for creation and update payloads when not all fields are required.
- Concrete repositories extend `BaseRepository<SpecificEntity>` and add domain-specific methods.
- Return domain types (`T`), not raw database rows, to keep the abstraction meaningful.

**Constraints and limitations.**

- Generic repositories can accumulate query-shaped methods over time, becoming a “god object”.
- Microsoft’s EF Core guidance explicitly recommends **against** wrapping `DbContext` in a generic repository in the common case, because it can obscure the ORM’s capabilities.
- Repository methods that use `Partial<T>` do not enforce required fields at the type level; runtime validation is still needed.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic in-memory repository

```typescript
// example_repository.ts
// Step 1: Define an entity constraint.
interface Entity {
  id: string;
}

// Step 2: Define the generic repository interface.
interface IRepository<T extends Entity> {
  findById(id: string): T | undefined;
  findAll(): T[];
  create(data: Omit<T, "id"> & { id?: string }): T;
  update(id: string, update: Partial<T>): T | undefined;
  delete(id: string): boolean;
}

// Step 3: Implement a generic base repository.
class InMemoryRepository<T extends Entity> implements IRepository<T> {
  protected items: T[] = [];

  findById(id: string): T | undefined {
    return this.items.find((item) => item.id === id);
  }

  findAll(): T[] {
    return [...this.items];
  }

  create(data: Omit<T, "id"> & { id?: string }): T {
    const item = { ...data, id: data.id ?? crypto.randomUUID() } as T;
    this.items.push(item);
    return item;
  }

  update(id: string, update: Partial<T>): T | undefined {
    const index = this.items.findIndex((item) => item.id === id);
    if (index === -1) return undefined;
    this.items[index] = { ...this.items[index], ...update };
    return this.items[index];
  }

  delete(id: string): boolean {
    const index = this.items.findIndex((item) => item.id === id);
    if (index === -1) return false;
    this.items.splice(index, 1);
    return true;
  }
}

// Step 4: Define a User entity and a concrete repository.
interface User extends Entity {
  name: string;
  email: string;
}

class UserRepository extends InMemoryRepository<User> {
  findByEmail(email: string): User | undefined {
    return this.items.find((user) => user.email === email);
  }
}

// Step 5: Use the repository with full type safety.
const users = new UserRepository();
const ada = users.create({ name: "Ada", email: "ada@example.com" });
console.log(users.findById(ada.id)?.name); // Expected: Ada
console.log(users.findByEmail("ada@example.com")?.email); // Expected: ada@example.com
```

**Expected Output:**
```
Ada
ada@example.com
```

**Why this output occurs.** `InMemoryRepository<User>` is parameterized by `User`, so `findById` returns `User | undefined`, and `create` accepts `Omit<User, "id">`. `UserRepository` adds `findByEmail`, which accesses `this.items` (typed as `User[]`). The generic constraint `T extends Entity` ensures `id` exists for all operations.

---

#### Example 2: Repository with identity constraint and partial updates

```typescript
// example_repository_constraint.ts
// Step 1: Constrain T to have a numeric id.
interface Persisted {
  id: number;
}

// Step 2: Generic repository with constrained T.
class Repository<T extends Persisted> {
  private store = new Map<number, T>();

  save(entity: T): void {
    this.store.set(entity.id, entity);
  }

  get(id: number): T | undefined {
    return this.store.get(id);
  }

  update(id: number, patch: Partial<Omit<T, "id">>): T | undefined {
    const existing = this.store.get(id);
    if (!existing) return undefined;
    const updated = { ...existing, ...patch } as T;
    this.store.set(id, updated);
    return updated;
  }
}

// Step 3: Use with a concrete entity.
interface Product extends Persisted {
  name: string;
  price: number;
}

const repo = new Repository<Product>();
repo.save({ id: 1, name: "Keyboard", price: 99 });

const updated = repo.update(1, { price: 89 });
console.log(updated?.price); // Expected: 89
```

**Expected Output:**
```
89
```

**Why this output occurs.** The constraint `T extends Persisted` guarantees that every entity has a numeric `id`, enabling `Map<number, T>`. `Partial<Omit<T, "id">>` prevents the `id` from being changed in an update, which is a common repository invariant. The type system enforces both the identity requirement and the update-shape restriction.

---

### Real-World Cases

- **NestJS/TypeORM**: `Repository<Entity>` is a built-in generic repository; concrete repositories extend it for domain-specific queries.
- **Drizzle repositories**: code generation produces generic base classes with custom query extensibility.
- **MongoDB/Mongoose**: `BaseRepository<T extends Document>` wraps Mongoose models with typed CRUD operations.
- **DDD data layers**: repository interfaces isolate domain logic from persistence, enabling in-memory test doubles.

---

## 4. Builder Pattern (Generic)

### Definitions

**Core Definition.** A **generic builder pattern** is a creational pattern in which a builder class or function is parameterized by the type of object being built (and often by the accumulated state of the builder), using chained method calls to incrementally construct a complex object while tracking which fields have been set at the type level.

**Technical Definition.** A generic builder declares a type parameter representing the **accumulated state** (the set of fields configured so far). Each configuration method returns a **new builder type** with the state extended via intersection types (`TState & { field: Type }`). Terminal methods (e.g., `build()`) use a `this` parameter constrained to require the necessary state, making it a compile-time error to build an incomplete object.

**Beginner-Friendly Explanation.** A generic builder is **a step-by-step object assembler that remembers what you have already set**. Each method call (`.host(...)`, `.port(...)`) updates the builder’s “memory” at the type level. The `build()` method only becomes available once all required fields are set. If you forget a required field, TypeScript gives you an error **before you run the code** — the builder literally will not let you call `build()`.

### Purposes

- **To construct complex objects step by step** without telescoping constructors.
- **To enforce required-field invariants at compile time** so incomplete objects cannot be built.
- **To preserve literal type information** for configured values (e.g., table names, column lists) in the final type.
- **To provide a fluent, chainable API** that is both ergonomic and type-safe.
- **To avoid the partial-inference problem** by binding type information incrementally across method calls.

### Syntax Rules and Structure

**General syntax (type-state builder):**

```
class Builder<TState extends BaseState> {
  private state: TState;

  method<V>(value: V): Builder<TState & { field: V }> {
    // update state and return new builder type
  }

  build(this: Builder<TState & RequiredState>): Result {
    // ...
  }
}
```

**Component breakdown:**

- `Builder<TState extends BaseState>` — the builder class parameterized by accumulated state.
- `method<V>(value: V): Builder<TState & { field: V }>` — a configuration method that returns a builder with extended state.
- `TState & { field: V }` — **intersection type** that adds the configured field to the accumulated state.
- `build(this: Builder<TState & RequiredState>)` — a terminal method whose `this` parameter requires the necessary state, making it uncallable on incomplete builders.

**Syntax rules.**

- Each configuration method should return a **new builder type**, not `this`, to accumulate state.
- Use intersection (`&`) to add fields without losing previously accumulated state.
- The `this` parameter on terminal methods enforces required fields at compile time.
- Capture literal types with a generic constrained to `string` (e.g., `<Id extends string>(id: Id)`) to preserve precise keys.
- Immutable builders should return new instances; mutable builders may cast internally but should preserve the external type contract.

**Constraints and limitations.**

- Type-state builders can produce complex type errors that are difficult for beginners to interpret.
- Internal casts (`as any`) are sometimes needed because the runtime state and compile-time state diverge.
- Overly complex type-state tracking can slow down the compiler on very large builders.
- The builder pattern is not always superior to an options object with optional fields; use it when step-wise invariants genuinely matter.

### Annotated Complete Step by Step Code Examples

#### Example 1: Database seeder builder with accumulated type state

```typescript
// example_builder_seeder.ts
// Step 1: Define base entity types.
interface User {
  id: string;
  name: string;
}
interface Post {
  id: string;
  title: string;
  authorId: string;
}

// Step 2: Define the database shape constraint.
interface DbShape {
  users: Record<string, User>;
  posts: Record<string, Post>;
}

// Step 3: Create the generic builder tracking accumulated state.
class DbSeeder<TDatabase extends DbShape> {
  public users: DbShape["users"] = {};
  public posts: DbShape["posts"] = {};

  addUser = <Id extends string>(
    id: Id,
    user: Omit<User, "id">
  ): DbSeeder<TDatabase & { users: TDatabase["users"] & Record<Id, User> }> => {
    this.users[id] = { ...user, id };
    return this;
  };

  addPost = <Id extends string>(
    id: Id,
    post: Omit<Post, "id">
  ): DbSeeder<TDatabase & { posts: TDatabase["posts"] & Record<Id, Post> }> => {
    this.posts[id] = { ...post, id };
    return this;
  };

  transact = async () => {
    return {
      users: this.users as TDatabase["users"],
      posts: this.posts as TDatabase["posts"],
    };
  };
}

// Step 4: Use with chained calls; each call extends the type.
const run = async () => {
  const result = await new DbSeeder()
    .addUser("matt", { name: "Matt" })
    .addPost("post1", { authorId: "matt", title: "Hello" })
    .addPost("post2", { authorId: "matt", title: "World" })
    .transact();

  console.log(result.users.matt.name); // Expected: Matt
  console.log(result.posts.post1.title); // Expected: Hello
  console.log(result.posts.post2.title); // Expected: World
};
run();
```

**Expected Output:**
```
Matt
Hello
World
```

**Why this output occurs.** Each `addUser` and `addPost` call extends `TDatabase` with a literal key (`"matt"`, `"post1"`, `"post2"`). After the chain, `TDatabase` includes `users: Record<"matt", User>` and `posts: Record<"post1" | "post2", Post>`. The `transact` method returns the accumulated shape, so `result.users.matt` is typed as `User` and `result.posts.post1` is typed as `Post`.

---

#### Example 2: Query builder with required `from()` before `build()`

```typescript
// example_builder_query.ts
// Step 1: Define the query state.
interface QueryState {
  table: string | null;
  columns: string[];
  whereClause: string | null;
}

// Step 2: Create the type-state query builder.
class QueryBuilder<TState extends QueryState> {
  private constructor(private state: TState) {}

  static create() {
    return new QueryBuilder({ table: null, columns: [], whereClause: null });
  }

  from<T extends string>(table: T): QueryBuilder<TState & { table: T }> {
    return new QueryBuilder({ ...this.state, table });
  }

  select<C extends string[]>(...columns: C): QueryBuilder<TState & { columns: C }> {
    return new QueryBuilder({ ...this.state, columns });
  }

  where<W extends string>(clause: W): QueryBuilder<TState & { whereClause: W }> {
    return new QueryBuilder({ ...this.state, whereClause: clause });
  }

  // build() is only available when table is set.
  build(this: QueryBuilder<TState & { table: string }>): string {
    const cols = this.state.columns.length ? this.state.columns.join(", ") : "*";
    let sql = `SELECT ${cols} FROM ${this.state.table}`;
    if (this.state.whereClause) {
      sql += ` WHERE ${this.state.whereClause}`;
    }
    return sql;
  }
}

// Step 3: Valid usage.
const sql = QueryBuilder.create()
  .from("users")
  .select("id", "name")
  .where("active = true")
  .build();

console.log(sql);
// Expected: SELECT id, name FROM users WHERE active = true

// Step 4: Invalid usage — build() without from() is a compile-time error.
// QueryBuilder.create().select("id").build();
// Error: The 'this' context of type 'QueryBuilder<...>' is not assignable to
//        method's 'this' of type 'QueryBuilder<... & { table: string }>'.
```

**Expected Output:**
```
SELECT id, name FROM users WHERE active = true
```

**Why this output occurs.** The `build` method’s `this` parameter requires `TState & { table: string }`. After `.from("users")`, `TState` includes `table: "users"`, so `build` is callable. Without `.from()`, the builder’s state lacks `table`, and the compiler rejects the `build` call. This is **compile-time enforcement of a required field** using the type system alone.

---

### Real-World Cases

- **SQL/query builders**: Knex.js and Drizzle use fluent, type-safe builders that track table and column selections.
- **Configuration builders**: server configuration objects where `host` and `port` must be set before `build()`.
- **Test data seeders**: the `DbSeeder` pattern shown above is used to seed databases with type-safe entity references.
- **Fluent API design**: libraries such as `fluent-builder` generate type-safe builders from schemas.

---

## 5. Generic Pattern Best Practices

### Definitions

**Core Definition.** **Generic pattern best practices** are guidelines for designing generic patterns that are **safe, ergonomic, inference-friendly, and maintainable**, avoiding over-parameterization, unnecessary complexity, and type-safety pitfalls.

**Technical Definition.** Best practices center on the principles of **minimal type parameters**, **constraint-driven safety**, **inference-first design**, and **type-state correctness**. Official TypeScript guidance states: “Always use as few type parameters as possible” and warns against generic types that do not use their type parameters. Well-designed generic patterns let callers omit type arguments through inference, constrain type parameters to exactly what is needed, and use type-level state only when the invariant genuinely matters.

**Beginner-Friendly Explanation.** Best practices mean **do not overcomplicate your generics**. Use the smallest number of type parameters that solves the problem. Let TypeScript guess the types from your values instead of forcing callers to write `<string>` everywhere. Only add type-level complexity — like the builder’s type-state tracking — when it prevents real bugs. If a generic does not make the API safer or easier to use, it is probably not worth adding.

### Purposes

- **To maximize type safety without sacrificing ergonomics**.
- **To keep generic APIs intuitive and inference-friendly**, so callers rarely need explicit type arguments.
- **To avoid unused type parameters** that add noise without benefit.
- **To constrain type parameters precisely**, preventing invalid instantiations while avoiding over-restriction.
- **To apply type-state complexity only where it prevents concrete bugs**, such as required-field enforcement.

### Syntax Rules and Structure

Best practices are conventions that interact with generic syntax. The core rules are:

1. **Use the fewest type parameters possible.** If a type parameter appears only once, it is likely unnecessary.
2. **Never declare a type parameter that is not used** in the signature.
3. **Design for inference, not annotation.** Callers should not need to specify type arguments in common cases.
4. **Constrain precisely.** Use `extends` to restrict to the exact capabilities needed, not broad types.
5. **Use `unknown` over `any`** when a type is genuinely open-ended.
6. **Reserve type-state tracking for genuine invariants**, such as required fields or accumulated configuration.
7. **Prefer simple options objects** when step-wise invariants are not required; do not force the Builder pattern where it adds no value.

**Component breakdown (the “minimal generics” checklist):**

```
Is a type parameter necessary?
  ├── No → Use a concrete type or union.
  └── Yes → Does it appear in the signature?
            ├── No → Remove it.
            └── Yes → Is it constrained as narrowly as possible?
                      ├── No → Tighten the constraint.
                      └── Yes → Can callers rely on inference?
                                ├── No → Adjust parameter positions.
                                └── Yes → Keep it.
```

**Constraints and limitations.**

- Over-constraining can make a generic pattern unusable with legitimate types.
- Under-constraining forces unsafe casts inside the pattern.
- Type-state builders are powerful but can produce complex errors; document them clearly.
- Generic repositories can become bloated if every query is added to the base class; prefer concrete extensions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Reducing unnecessary type parameters

```typescript
// example_best_fewer.ts
// ❌ Over-parameterized: U is unused in the signature.
function badLog<T, U>(value: T): void {
  console.log(value);
}

// ✅ Minimal: only T is needed.
function goodLog<T>(value: T): void {
  console.log(value);
}

// Both work, but goodLog is simpler and inference-friendly.
goodLog("hello"); // Expected: hello
goodLog(42);      // Expected: 42
```

**Expected Output:**
```
hello
42
```

**Why this output occurs.** `badLog` declares an unused type parameter `U`, which adds noise and can confuse inference. `goodLog` uses only `T`, which is inferred from the argument. The simplified signature is easier to read and equally type-safe. This aligns with the guideline to use the fewest type parameters possible.

---

#### Example 2: Precise constraints instead of broad `any`

```typescript
// example_best_constraint.ts
// ❌ Broad: no constraint, unsafe property access required.
function badGetId<T>(entity: T): string {
  return (entity as any).id;
}

// ✅ Constrained: T must have an id.
function goodGetId<T extends { id: string }>(entity: T): string {
  return entity.id;
}

const user = { id: "u1", name: "Ada" };
console.log(goodGetId(user)); // Expected: u1
```

**Expected Output:**
```
u1
```

**Why this output occurs.** The unconstrained version requires an unsafe cast to access `id`. The constrained version guarantees that `T` has an `id: string`, so `entity.id` is statically safe and the return type is accurate. Constraints should be as narrow as necessary but no narrower.

---

#### Example 3: Inference-friendly factory design

```typescript
// example_best_inference.ts
// ✅ The type parameter is inferred from the constructor argument.
function create<T>(ctor: new () => T): T {
  return new ctor();
}

class Service {
  ping() {
    return "pong";
  }
}

const service = create(Service); // T inferred as Service
console.log(service.ping());     // Expected: pong
```

**Expected Output:**
```
pong
```

**Why this output occurs.** The caller does not need to write `create<Service>(Service)`. The type parameter `T` is inferred from the constructor argument, so the return type is `Service`. Designing for inference means placing type parameters where they can be derived from values, rather than forcing explicit annotation.

---

### Real-World Cases

- **Library API design**: successful libraries (TanStack Query, Zod, tRPC) design generics so simple usage requires no type arguments.
- **Enterprise architecture**: constrained generic repositories (`T extends { id: string }`) prevent data-layer type drift.
- **Test utilities**: generic factories and builders reduce fixture duplication while preserving entity types.
- **Type-safe DSLs**: query builders use type-state tracking only for genuine invariants (e.g., `from()` before `build()`), avoiding unnecessary complexity.

---

## References

- TypeScript Handbook — Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook — Do’s and Don’ts (Generics) - https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html
- Microsoft/TypeScript Wiki — FAQ (Why doesn’t type inference work on this interface?) - https://github.com/microsoft/TypeScript/wiki/FAQ
- TypeScript Handbook — Utility Types (`Partial`, `Omit`, `Record`, `InstanceType`) - https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Design Patterns: SOLID Principles and Type-Safe Architecture - https://dakic.com/topics/typescript/typescript-design-patterns-solid-type-safe-architecture/
- Builder Pattern — mcollina/skills (TypeScript Magician) - https://github.com/mcollina/skills/blob/main/skills/typescript-magician/rules/builder-pattern.md
- How to Implement the Repository Pattern with MongoDB (OneUptime Blog) - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-mongodb-repository-pattern/README.md
- How to maintain type safety with a factory pattern and generic base class? — Stack Overflow - https://stackoverflow.com/questions/78732161
- TypeScript Patterns — Builder, Currying, Factory with Generics - https://raw.githubusercontent.com/tshemsedinov/Patterns-JavaScript/master/README.md
- Total TypeScript — The Power of Generics and the Builder Pattern - https://www.totaltypescript.com/workshops/type-transformations/type-helpers/optional-type-parameters-in-type-helpers/solution