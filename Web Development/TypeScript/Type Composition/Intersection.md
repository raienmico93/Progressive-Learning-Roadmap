# TypeScript Intersection Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An intersection type in TypeScript is a type that combines two or more types into one, requiring a value to satisfy *all* constituent types simultaneously. Intersection types are written using the ampersand operator (`&`) between types. A value of an intersection type has all the members of every constituent type.

**Technical Definition**
An intersection type is a TypeScript type construct that denotes the set-theoretic intersection of its constituent types. Given types `A` and `B`, the intersection `A & B` contains all values that are members of both `A` and `B`. For object types, this means the resulting type has all properties from both `A` and `B`. When property types conflict between constituents, the intersection of those property types is computed (e.g., `string & number` results in `never` because no value is both a string and a number). For callable types (functions), intersection produces overloaded signatures. Intersection types are the primary mechanism for composing behaviors and mixing properties in TypeScript, and they serve as the type-level counterpart to the mixin pattern.

**Beginner-Friendly Explanation**
An intersection type says "this value must be *all* of these types at once." If you have `Person & Employee`, a value of that type must have all of `Person`'s properties *and* all of `Employee`'s properties. It's like combining two checklists into one—you have to check off everything from both. Intersection types are written with `&` (ampersand). They're used to combine object shapes, add properties to existing types, and compose interfaces. Unlike union types (which say "one of these"), intersection types say "all of these."

### Key Characteristics

- **Set-theoretic intersection**: A value must satisfy *all* constituent types.
- **Property merging**: Object intersections merge all properties from all members.
- **Conflict resolution**: Conflicting primitive property types result in `never`.
- **Method overloading**: Conflicting method signatures produce overloads.
- **Composition tool**: Intersections are the primary way to compose object types.
- **Mixins**: Intersections model the type-level result of mixin patterns.
- **Structural**: Intersection compatibility is structural.
- **Compile-time only**: Erased at runtime.

### Prerequisites

- Basic knowledge of TypeScript object types and interfaces
- Familiarity with union types (`|`)
- Understanding of type aliases and interfaces
- Familiarity with generics and utility types

### Related Programming Areas

- **Type Theory**: Intersection types are product types (in the type-theoretic sense) for object members
- **Design Patterns**: Mixin, decorator, and adapter patterns
- **Interface Composition**: Intersections compose interfaces without inheritance
- **Utility Types**: Many built-in utility types (`Readonly<T>`, `Partial<T>`) use intersections internally
- **Module Augmentation**: Intersections extend third-party types

### Core Concepts / Features

1. Intersection Syntax (`&`) and Conceptual Meaning
2. Combining Object Types and Mixing Properties
3. Composition of Interfaces with Intersection Types
4. Conflicting Properties (Primitive Mismatches Resulting in `never` vs. Method Overloading)
5. Practical Use Cases (Mixins, Extending Third-Party Library Options)


## 1. Intersection Syntax (`&`) and Conceptual Meaning

### Definitions

**Core Definition**
Intersection syntax uses the ampersand operator (`&`) to combine two or more types into a single type that requires values to satisfy all constituent types. The conceptual meaning is "all of these types" or "this AND that."

**Technical Definition**
The intersection type operator (`&`) is a binary type operator that constructs the intersection of its operands. `A & B` is the type whose values are exactly the values that are members of *both* `A` and `B`. For object types, the resulting type has the union of all properties from both operands (with property type intersections applied for conflicting names). For primitive types, `A & B` is `never` unless one is a subtype of the other (in which case, the subtype wins). For callable types, the intersection produces overloaded signatures. TypeScript normalizes intersections: duplicate members are merged, `never` absorbs the entire intersection (`T & never` is `never`), `unknown` is the identity (`T & unknown` is `T`), and `any` absorbs everything (`T & any` is `any`).

**Beginner-Friendly Explanation**
The ampersand (`&`) is TypeScript's way of saying "and." When you write `Person & Employee`, you're saying "this value must have everything a Person has AND everything an Employee has." It's like combining two requirements into one. Intersection types are useful when you want to compose behaviors: `Timestamped & Serializable` gives you a type that's both timestamped and serializable. Unlike union types (which are "or"), intersections are "and."

### Purposes

- To combine multiple object types into a single type with all members.
- To compose interfaces without using inheritance.
- To add properties to an existing type without modifying it.
- To model mixin patterns at the type level.
- To constrain generic type parameters to multiple contracts simultaneously.

### Syntax Rules and Structure

**General Syntax: Intersection Type**

```typescript
type AliasName = Type1 & Type2 & Type3;
```

**Component Breakdown**
- `Type1 & Type2 & Type3`: The intersection of the constituent types.
- A value of `AliasName` must satisfy all three types.

**General Syntax: Inline Intersection Annotation**

```typescript
let variable: Type1 & Type2 = value;
```

**Component Breakdown**
- The intersection can be used directly in variable, parameter, or return type positions.

**General Syntax: Intersection in Generic Constraints**

```typescript
function functionName<T extends Type1 & Type2>(value: T): void { }
```

**Component Breakdown**
- `T extends Type1 & Type2`: The generic parameter must satisfy both types.

**Syntax Rules**

- The `&` operator joins two or more types into an intersection.
- Intersection members can be any valid type: primitives, objects, literals, functions, tuples.
- Duplicate members are merged.
- `never` absorbs the intersection (`T & never` is `never`).
- `unknown` is the identity (`T & unknown` is `T`).
- `any` absorbs the intersection (`T & any` is `any`).
- Object intersections merge all properties.
- Conflicting primitive properties become `never`.
- Conflicting method signatures become overloads.

**Constraints and Limitations**

- Intersections of incompatible primitives (`string & number`) produce `never`, which is rarely useful.
- Intersection types are erased at runtime; they exist only in the type system.
- Complex intersections can be hard to read; consider extracting to named interfaces.
- Intersection type members cannot be overridden—they accumulate.
- Intersections do not create nominal identity; two structurally identical intersections are compatible.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Intersection Types

```typescript
// Step 1: Define two interfaces.
interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: number;
  department: string;
}

// Step 2: Create an intersection type.
type EmployeePerson = Person & Employee;

// Step 3: Create a value satisfying both.
const alice: EmployeePerson = {
  name: "Alice",
  age: 30,
  employeeId: 1001,
  department: "Engineering",
};

// Step 4: All properties are accessible.
console.log(`${alice.name} (ID: ${alice.employeeId})`);
console.log(`Department: ${alice.department}, Age: ${alice.age}`);

// Step 5: Missing properties from either interface cause errors.
// const bob: EmployeePerson = {
//   name: "Bob",
//   age: 25,
//   employeeId: 1002,
//   // department is missing
// };
// ❌ Error: Property 'department' is missing.

// Step 6: Both interfaces are satisfied.
function describe(person: Person): string {
  return `${person.name}, ${person.age}`;
}

function describeEmployee(employee: Employee): string {
  return `${employee.employeeId} in ${employee.department}`;
}

console.log(describe(alice));          // "Alice, 30"
console.log(describeEmployee(alice));  // "1001 in Engineering"
```

**Expected Output:**
```
Alice (ID: 1001)
Department: Engineering, Age: 30
Alice, 30
1001 in Engineering
```

**Why This Output Occurs:** The `EmployeePerson` intersection requires all properties from both `Person` and `Employee`. The `alice` object satisfies both, so it can be used wherever either `Person` or `Employee` is expected.

#### Example 2: Intersection Normalization

```typescript
// Step 1: TypeScript normalizes intersections.
type A = { name: string } & { name: string };         // Merges duplicate
type B = { name: string } & unknown;                  // unknown is identity
type C = { name: string } & any;                      // any absorbs everything
type D = { name: string } & never;                    // never absorbs everything

// Step 2: Verify normalization.
const a: A = { name: "Alice" };  // ✅
const b: B = { name: "Bob" };    // ✅
const c: C = { name: "Charlie" }; // ✅ (but c is `any`)

// Step 3: never intersection makes the type uninhabitable.
// const d: D = { name: "Dave" };
// ❌ Error: Type '{ name: string; }' is not assignable to type 'never'.

// Step 4: Intersection with any loses type safety.
function useAny(value: C): void {
  console.log(value.nonExistent.deeply.nested);  // ✅ No error (value is any)
}

useAny({ name: "Test" });  // Runtime error if nonExistent accessed
```

**Expected Output:** No output (the `useAny` call would produce a runtime error if the property chain were actually accessed; the code is shown to compile).

**Why This Output Occurs:** TypeScript normalizes intersections by merging duplicates and applying identity/absorbing rules. `unknown` is the identity element (`T & unknown = T`). `any` absorbs everything (`T & any = any`). `never` absorbs everything (`T & never = never`), making the type uninhabitable.

### Real-World Cases

**Case 1: Mixin Composition**
Mixins in TypeScript are often modeled with intersection types: `Timestamped & Serializable & Loggable`.

**Case 2: Utility Types**
TypeScript's built-in utility types use intersections. For example, `Required<T>` is implemented as a mapped type that intersects with the original.

**Case 3: Third-Party Augmentation**
Intersections are used to add properties to third-party types without modifying the source.

---

## 2. Combining Object Types and Mixing Properties

### Definitions

**Core Definition**
Combining object types with intersections merges the properties of all constituent object types into a single type. The resulting type has every property from every constituent, with property types computed via intersection when names conflict.

**Technical Definition**
For object types `A` and `B`, the intersection `A & B` produces a type with the union of all property names from `A` and `B`. For each property name, the resulting type is `A[P] & B[P]` if the property exists in both, or the sole type if it exists in only one. Property modifiers are merged: a property is optional in the intersection only if it is optional in all constituents; a property is readonly if it is readonly in any constituent. This merging enables powerful composition patterns: combining base types, adding capabilities, and layering concerns.

**Beginner-Friendly Explanation**
When you intersect two object types, you get a type with all properties from both. For example, `{ name: string } & { age: number }` produces `{ name: string; age: number }`. It's like combining two Lego pieces into one. If both types have the same property with the same type, it stays the same. If they have the same property with different types, TypeScript intersects the types (which usually results in `never`). Intersections are the cleanest way to compose object shapes without inheritance.

### Purposes

- To merge properties from multiple object types into one.
- To add capabilities to a base type without modifying it.
- To compose domain models from smaller, focused interfaces.
- To implement mixin-style type composition.
- To constrain generic types to multiple object contracts.

### Syntax Rules and Structure

**General Syntax: Object Intersection**

```typescript
type Combined = { prop1: Type1 } & { prop2: Type2 };
// Result: { prop1: Type1; prop2: Type2 }
```

**Component Breakdown**
- The intersection merges the properties of both object literals.

**General Syntax: Interface Intersection**

```typescript
interface A { a: string; }
interface B { b: number; }
type C = A & B;
// Result: { a: string; b: number }
```

**Component Breakdown**
- Intersections work with named interfaces as well as inline object types.

**General Syntax: Optional and Readonly Merging**

```typescript
type A = { prop?: string };
type B = { prop: string };
type C = A & B;  // prop is required in C (required wins)
```

**Component Breakdown**
- A property is optional only if optional in all constituents.
- A property is readonly if readonly in any constituent.

**Syntax Rules**

- Object intersections merge all properties from all constituents.
- Property types are intersected for conflicting names.
- Optionality: a property is optional only if optional in all constituents.
- Readonly: a property is readonly if readonly in any constituent.
- Extra properties on object literals are subject to excess property checking.
- Intersections of object types are commutative and associative (order doesn't matter).
- Intersections can be combined with unions (with careful precedence).

**Constraints and Limitations**

- Conflicting primitive property types result in `never` (rarely useful).
- Intersection types cannot override property types (only intersect them).
- Deeply nested intersections can be complex to read.
- Excess property checking with intersections can be surprising.
- Intersections of object types with methods can produce overloads if signatures conflict.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Merging Object Properties

```typescript
// Step 1: Define base object types.
type HasName = { name: string };
type HasAge = { age: number };
type HasEmail = { email: string };

// Step 2: Combine with intersections.
type Person = HasName & HasAge;
type Contact = HasName & HasEmail;
type FullProfile = HasName & HasAge & HasEmail;

// Step 3: Create values.
const person: Person = { name: "Alice", age: 30 };
const contact: Contact = { name: "Bob", email: "bob@example.com" };
const profile: FullProfile = { name: "Charlie", age: 25, email: "charlie@example.com" };

console.log(person);   // { name: 'Alice', age: 30 }
console.log(contact);  // { name: 'Bob', email: 'bob@example.com' }
console.log(profile);  // { name: 'Charlie', age: 25, email: 'charlie@example.com' }

// Step 4: Each intersection is assignable to its constituents.
function greet(p: HasName): string {
  return `Hello, ${p.name}`;
}

console.log(greet(person));   // "Hello, Alice"
console.log(greet(contact));  // "Hello, Bob"
console.log(greet(profile));  // "Hello, Charlie"

// Step 5: Missing properties cause errors.
// const invalid: FullProfile = { name: "Dave", age: 40 };
// ❌ Error: Property 'email' is missing.
```

**Expected Output:**
```
{ name: 'Alice', age: 30 }
{ name: 'Bob', email: 'bob@example.com' }
{ name: 'Charlie', age: 25, email: 'charlie@example.com' }
Hello, Alice
Hello, Bob
Hello, Charlie
```

**Why This Output Occurs:** Intersections merge all properties. `FullProfile` has `name`, `age`, and `email`. Values of `FullProfile` are assignable to `HasName`, `HasAge`, and `HasEmail` because they have all required properties.

#### Example 2: Optional and Readonly Merging

```typescript
// Step 1: Define types with modifiers.
type WithOptional = { nickname?: string; bio?: string };
type WithRequired = { nickname: string; email: string };
type WithReadonly = { readonly id: number; name: string };

// Step 2: Intersect types with different modifiers.
type MergedOptionalRequired = WithOptional & WithRequired;
// nickname: string (required — because WithRequired requires it)
// bio?: string (optional — only in WithOptional)
// email: string (required — from WithRequired)

const merged: MergedOptionalRequired = {
  nickname: "Al",
  email: "al@example.com",
  // bio is optional, so it can be omitted
};
console.log(merged);  // { nickname: 'Al', email: 'al@example.com' }

// Step 3: Readonly merging.
type MergedReadonly = WithReadonly & { name: string };
// id: readonly number (readonly wins)
// name: string (mutable)

const item: MergedReadonly = { id: 1, name: "Widget" };
// item.id = 2;  // ❌ Error: Cannot assign to 'id' because it is a read-only property.
item.name = "Gadget";  // ✅ Allowed
console.log(item);  // { id: 1, name: 'Gadget' }
```

**Expected Output:**
```
{ nickname: 'Al', email: 'al@example.com' }
{ id: 1, name: 'Gadget' }
```

**Why This Output Occurs:** When merging optional with required, required wins (the property must be present). When merging readonly with mutable, readonly wins (the property cannot be reassigned). These rules ensure that the intersection is the *most restrictive* combination.

### Real-World Cases

**Case 1: Domain Modeling**
Domain models combine base traits: `type User = HasId & HasTimestamps & HasName & HasEmail`.

**Case 2: API DTOs**
API DTOs combine request-specific fields with common metadata: `type CreateUserRequest = UserFields & AuditFields`.

**Case 3: UI Props**
UI component props combine base props with component-specific props: `type ButtonProps = BaseProps & { variant: "primary" | "secondary" }`.

---

## 3. Composition of Interfaces with Intersection Types

### Definitions

**Core Definition**
Intersection types compose interfaces by combining their members into a single type. This provides an alternative to interface inheritance (`extends`) for composing contracts, with the key difference that intersections do not create a subtype relationship—they create a new type with all members.

**Technical Definition**
Interface composition with intersections merges the members of multiple interfaces. Unlike `interface C extends A, B`, which creates a subtype relationship where `C` is assignable to `A` and `B`, the intersection `type C = A & B` creates a type that is assignable to both `A` and `B` and has all their members. The key difference is that interfaces can be extended (declaration merging, additional `extends`), while type aliases with intersections are fixed. However, intersections can combine interfaces that cannot be extended (e.g., interfaces with conflicting members, or interfaces combined with type aliases representing unions).

**Beginner-Friendly Explanation**
Intersections let you combine interfaces without inheritance. If you have `Readable` and `Writable` interfaces, you can create `ReadWritable = Readable & Writable`. The result has all members of both. This is similar to `interface ReadWritable extends Readable, Writable`, but with an important difference: intersections create a new type alias, not an interface. This means you can't extend the result further, but you *can* combine interfaces with type aliases (which `extends` doesn't allow). Intersections are a flexible composition tool.

### Purposes

- To compose interfaces without creating inheritance relationships.
- To combine interfaces with type aliases (which `extends` cannot do).
- To create "flattened" types from multiple interface contracts.
- To add capabilities to interfaces without modifying them.
- To build complex types from small, focused interfaces.

### Syntax Rules and Structure

**General Syntax: Interface Composition via Intersection**

```typescript
interface Readable { read(): string; }
interface Writable { write(data: string): void; }
type ReadWritable = Readable & Writable;
```

**Component Breakdown**
- `Readable & Writable`: The composed type with all members.

**General Syntax: Composition with Type Aliases**

```typescript
type HasId = { id: number };
type HasTimestamps = { createdAt: Date; updatedAt: Date };
type Entity = HasId & HasTimestamps;
```

**Component Breakdown**
- Type aliases (including unions) can be intersected.

**General Syntax: Composition with Generics**

```typescript
type WithMeta<T> = T & { meta: { createdAt: Date; updatedAt: Date } };
```

**Component Breakdown**
- Generic intersections add capabilities to any type.

**Syntax Rules**

- Interfaces and type aliases can be combined via intersections.
- The result is a type alias, not an interface (cannot be extended with `extends`).
- The result has all members from all constituents.
- The result is assignable to each constituent.
- Intersections can be generic.
- Intersections with unions require careful precedence (use parentheses).

**Constraints and Limitations**

- Intersections of interfaces cannot be extended (they are type aliases).
- Intersections do not create nominal identity.
- Intersection type aliases cannot participate in declaration merging.
- Very large intersections can impact compiler performance.
- Intersections with unions require careful parenthesization.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Composing Interfaces

```typescript
// Step 1: Define focused interfaces.
interface Readable {
  read(): string;
}

interface Writable {
  write(data: string): void;
}

interface Closeable {
  close(): void;
}

// Step 2: Compose them with intersections.
type ReadWritable = Readable & Writable;
type Stream = Readable & Writable & Closeable;

// Step 3: Implement the composed type.
const stream: Stream = {
  read() { return "data"; },
  write(data: string) { console.log(`Writing: ${data}`); },
  close() { console.log("Closed"); },
};

console.log(stream.read());  // "data"
stream.write("hello");       // "Writing: hello"
stream.close();              // "Closed"

// Step 4: The composed type is assignable to each interface.
const readable: Readable = stream;
const writable: Writable = stream;
const closeable: Closeable = stream;

console.log(readable.read());  // "data"

// Step 5: Compose with type aliases.
type HasId = { id: number };
type HasMeta = { createdAt: Date };
type Entity = ReadWritable & HasId & HasMeta;

const entity: Entity = {
  id: 1,
  createdAt: new Date(),
  read: () => "entity data",
  write: (data) => console.log(`Entity write: ${data}`),
};

console.log(entity.id);  // 1
```

**Expected Output:**
```
data
Writing: hello
Closed
data
1
```

**Why This Output Occurs:** The `Stream` type is the intersection of `Readable`, `Writable`, and `Closeable`, so it has all their methods. The composed type is assignable to each constituent, enabling polymorphic use. Adding `HasId` and `HasMeta` further composes the type.

#### Example 2: Generic Composition and Union Interactions

```typescript
// Step 1: Define a generic composition utility.
type WithTimestamp<T> = T & { createdAt: Date; updatedAt: Date };
type WithSoftDelete<T> = T & { deletedAt: Date | null };

// Step 2: Compose multiple utilities.
type User = { id: number; name: string };
type AuditedUser = WithTimestamp<WithSoftDelete<User>>;

const user: AuditedUser = {
  id: 1,
  name: "Alice",
  createdAt: new Date(),
  updatedAt: new Date(),
  deletedAt: null,
};

console.log(user.name);       // "Alice"
console.log(user.deletedAt);  // null

// Step 3: Intersection with unions requires parentheses.
type Status = "active" | "inactive";
type TaggedStatus = { status: Status } & { updatedAt: Date };

const tagged: TaggedStatus = {
  status: "active",
  updatedAt: new Date(),
};

console.log(tagged.status);  // "active"

// Step 4: Incorrect precedence without parentheses.
// type Wrong = { status: Status } & { updatedAt: Date } | { extra: string };
// This parses as: ({ status: Status } & { updatedAt: Date }) | { extra: string }
// which is probably not what was intended.

// Step 5: Correct parenthesization for unions.
type Correct = ({ status: "active" } | { status: "inactive" }) & { updatedAt: Date };
```

**Expected Output:**
```
Alice
null
active
```

**Why This Output Occurs:** Generic intersections add capabilities to any type. The `WithTimestamp<WithSoftDelete<User>>` composition adds both timestamp and soft-delete fields to `User`. Parenthesization is required when intersecting with unions because `&` binds tighter than `|`.

### Real-World Cases

**Case 1: Domain Entity Composition**
Domain entities compose from traits: `type User = HasId & HasTimestamps & HasSoftDelete & UserFields`.

**Case 2: GraphQL Type Composition**
GraphQL schemas compose from fragments: `type UserWithPosts = UserFragment & PostsFragment`.

**Case 3: Configuration Schemas**
Configuration schemas compose base options with environment-specific ones: `type ProdConfig = BaseConfig & ProdOverrides`.

---

## 4. Conflicting Properties (Primitive Mismatches Resulting in `never` vs. Method Overloading)

### Definitions

**Core Definition**
When intersecting types have properties with the same name but different types, TypeScript intersects the property types. For primitive types, this usually results in `never` (since no value can be both a string and a number). For function types, the intersection produces an overloaded function that must satisfy both signatures.

**Technical Definition**
For a property `P` present in both `A` and `B` with types `PA` and `PB`, the intersection `A & B` gives `P` the type `PA & PB`. For primitive types, `string & number` is `never`, `string & "hello"` is `"hello"` (literal absorbed), and `boolean & true` is `true`. For object types, `PA & PB` recursively merges their members. For function types, `(x: A) => B & (x: C) => D` produces an overloaded signature that can be called with either `A` or `C` (but must satisfy both). This behavior makes intersections powerful for method overloading but problematic for primitive conflicts.

**Beginner-Friendly Explanation**
When two types have a property with the same name but different types, TypeScript tries to intersect those types. For primitives, this usually fails: a property can't be both a string and a number, so TypeScript makes it `never` (which means "no value is possible"). For functions, TypeScript creates an overloaded function that can handle both signatures. This means intersections are great for overloading methods but bad for conflicting primitive properties. If you're seeing `never` in your intersection, it's probably because two types have a property with incompatible primitive types.

### Purposes

- To understand why conflicting primitive properties produce `never`.
- To use method overloading via intersections.
- To diagnose and fix `never`-producing intersections.
- To compose function types with multiple signatures.
- To avoid unintended conflicts when combining object types.

### Syntax Rules and Structure

**General Syntax: Primitive Conflict**

```typescript
type A = { value: string };
type B = { value: number };
type C = A & B;
// value: string & number = never
```

**Component Breakdown**
- `string & number` is `never`, making `C` uninhabitable.

**General Syntax: Literal Absorption**

```typescript
type A = { value: string };
type B = { value: "hello" };
type C = A & B;
// value: string & "hello" = "hello"
```

**Component Breakdown**
- Literal types are absorbed by their supertypes.

**General Syntax: Method Overloading via Intersection**

```typescript
type A = { fn: (x: string) => string };
type B = { fn: (x: number) => number };
type C = A & B;
// fn: { (x: string): string; (x: number): number; }
```

**Component Breakdown**
- The intersection produces an overloaded function with both signatures.

**Syntax Rules**

- Primitive property conflicts produce `never` (unless one is a subtype of the other).
- Literal types are absorbed by their supertypes.
- Object property conflicts produce recursive intersections.
- Function property conflicts produce overloaded signatures.
- Optional properties remain optional only if optional in all constituents.
- Readonly properties remain readonly if readonly in any constituent.

**Constraints and Limitations**

- `never`-producing intersections are uninhabitable and rarely useful.
- Overloaded functions from intersections must satisfy both signatures.
- Conflicting method signatures can produce complex overload sets.
- The compiler may not always produce helpful error messages for `never` intersections.
- Intersection conflicts are resolved at compile time; no runtime error occurs.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Primitive Conflicts Resulting in `never`

```typescript
// Step 1: Define types with conflicting primitive properties.
type HasStringId = { id: string; name: string };
type HasNumberId = { id: number; email: string };

// Step 2: Intersect them — id becomes never.
type Conflicting = HasStringId & HasNumberId;
// id: string & number = never
// name: string
// email: string

// Step 3: Attempt to create a value — id is impossible.
// const invalid: Conflicting = {
//   id: "abc",  // ❌ Error: Type 'string' is not assignable to type 'never'.
//   name: "Alice",
//   email: "alice@example.com",
// };

// Step 4: The type is effectively uninhabitable due to `never`.
function process(value: Conflicting): void {
  // value.id is never — can't be assigned or used meaningfully
  console.log(value.name);   // ✅ Accessible
  console.log(value.email);  // ✅ Accessible
}

// Step 5: Literal absorption prevents some conflicts.
type HasBroadId = { id: string };
type HasLiteralId = { id: "user-123" };
type Absorbed = HasBroadId & HasLiteralId;
// id: string & "user-123" = "user-123"

const absorbed: Absorbed = { id: "user-123" };  // ✅ Allowed
console.log(absorbed.id);  // "user-123"
```

**Expected Output:**
```
user-123
```

**Why This Output Occurs:** The conflict between `string` and `number` produces `never`, making `Conflicting` uninhabitable. The conflict between `string` and the literal `"user-123"` is resolved by absorbing the literal into `string` (the literal is a subtype, so the intersection is the literal).

#### Example 2: Method Overloading via Intersection

```typescript
// Step 1: Define types with function properties.
type StringProcessor = {
  process: (input: string) => string;
};

type NumberProcessor = {
  process: (input: number) => number;
};

// Step 2: Intersect them — process becomes overloaded.
type MultiProcessor = StringProcessor & NumberProcessor;

// Step 3: Implement the overloaded function.
const processor: MultiProcessor = {
  process(input: string | number): string | number {
    if (typeof input === "string") {
      return input.toUpperCase();
    }
    return input * 2;
  },
};

// Step 4: Call with both types.
console.log(processor.process("hello"));  // "HELLO"
console.log(processor.process(21));       // 42

// Step 5: TypeScript enforces the overloads.
// processor.process(true);  // ❌ Error: No overload matches this call.

// Step 6: Multiple overloads compose.
type Logger = {
  log: (message: string) => void;
  log: (message: string, level: string) => void;  // Not valid — use intersection instead
};

type BaseLogger = { log: (message: string) => void };
type LeveledLogger = { log: (message: string, level: string) => void };
type FullLogger = BaseLogger & LeveledLogger;

const logger: FullLogger = {
  log(message: string, level?: string): void {
    console.log(level ? `[${level}] ${message}` : message);
  },
};

logger.log("Hello");              // "Hello"
logger.log("Warning", "WARN");    // "[WARN] Warning"
```

**Expected Output:**
```
HELLO
42
Hello
[WARN] Warning
```

**Why This Output Occurs:** The intersection of `StringProcessor` and `NumberProcessor` produces an overloaded `process` function with two signatures. The implementation accepts the union of parameter types and handles both. The `FullLogger` intersection produces an overloaded `log` function with one- and two-argument signatures.

### Real-World Cases

**Case 1: Event Handler Overloading**
Event systems use intersection types to type handlers that can accept different event shapes, producing overloaded call signatures.

**Case 2: Utility Function Overloading**
Libraries use intersections to type utility functions with multiple overloads (e.g., `lodash`'s `get` function).

**Case 3: API Client Methods**
API clients use intersections to type methods that accept different parameter combinations, enabling overloaded method signatures.

---

## 5. Practical Use Cases (Mixins, Extending Third-Party Library Options)

### Definitions

**Core Definition**
Intersection types are widely used in practice for mixin patterns, extending third-party library options, and composing complex types from simpler ones. The mixin pattern combines behaviors from multiple sources; intersection types model the resulting type. Third-party option extension adds application-specific options to library configurations without modifying the library.

**Technical Definition**
Mixin patterns in TypeScript use functions that take a base class and return a new class with additional behavior. The type-level representation of a mixed class is the intersection of the base class type and the mixin's contribution. Third-party option extension uses intersections to combine a library's option interface with application-specific options: `type MyOptions = LibraryOptions & { customField: string }`. This pattern is common with libraries like `passport`, `express`, and `webpack`, where developers augment configuration types without forking.

**Beginner-Friendly Explanation**
Intersections are used in two major real-world patterns. First, mixins: when you combine behaviors from multiple classes, the resulting type is an intersection. Second, extending library options: if a library defines `LibraryOptions`, you can create `MyOptions = LibraryOptions & { myCustomField: string }` to add your own options without modifying the library. These patterns make intersections one of the most practical TypeScript features—they let you compose and extend types cleanly.

### Purposes

- To model the type-level result of mixin composition.
- To add application-specific options to third-party library configurations.
- To compose complex types from small, reusable pieces.
- To extend existing types without modifying their definitions.
- To create domain-specific types on top of library types.

### Syntax Rules and Structure

**General Syntax: Mixin Type Composition**

```typescript
type MixedClass = BaseClass & Mixin1 & Mixin2;
```

**Component Breakdown**
- The intersection combines base class members with mixin members.

**General Syntax: Extending Third-Party Options**

```typescript
import { LibraryOptions } from "some-library";

type MyOptions = LibraryOptions & {
  customField: string;
  anotherOption?: number;
};
```

**Component Breakdown**
- The intersection adds custom fields to the library's options.

**General Syntax: Generic Mixin Utility**

```typescript
type WithLogging<T> = T & {
  log(message: string): void;
};

type LoggedUser = WithLogging<User>;
```

**Component Breakdown**
- Generic intersections add capabilities to any type.

**Syntax Rules**

- Mixin types are intersections of base and mixin contributions.
- Third-party options are extended via intersections with custom fields.
- Generic intersections can add capabilities to any type.
- Intersections preserve all members from all constituents.
- The composed type is assignable to each constituent.

**Constraints and Limitations**

- Mixins add runtime code; intersections only model types.
- Third-party option extensions must be compatible with the library's expectations.
- Excess property checking applies to intersections.
- Generic intersections can produce complex type errors.
- Intersections do not enforce that custom fields are actually used by the library.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Mixin Type Composition

```typescript
// Step 1: Define a base class.
class User {
  constructor(public name: string, public email: string) {}
}

// Step 2: Define mixin behaviors as functions.
function Timestamped<T extends new (...args: any[]) => {}>(Base: T) {
  return class extends Base {
    createdAt = new Date();
    updatedAt = new Date();
  };
}

function Loggable<T extends new (...args: any[]) => {}>(Base: T) {
  return class extends Base {
    log(message: string): void {
      console.log(`[LOG] ${message}`);
    }
  };
}

// Step 3: Apply mixins.
const TimestampedLoggableUser = Loggable(Timestamped(User));
const user = new TimestampedLoggableUser("Alice", "alice@example.com");

// Step 4: The instance has all members.
console.log(user.name);        // "Alice"
console.log(user.createdAt);   // Date
user.log("User created");      // "[LOG] User created"

// Step 5: The type-level composition is an intersection.
type UserWithMixins = User & { createdAt: Date; updatedAt: Date } & { log: (msg: string) => void };

const typedUser: UserWithMixins = user;  // ✅ Structurally compatible
console.log(typedUser.createdAt instanceof Date);  // true
```

**Expected Output:**
```
Alice
2024-...
[LOG] User created
true
```

**Why This Output Occurs:** The mixin functions add `createdAt`, `updatedAt`, and `log` to the `User` class. The resulting type is the intersection of `User` with the mixin contributions. The `UserWithMixins` type alias models this intersection, and the runtime instance is structurally compatible.

#### Example 2: Extending Third-Party Library Options

```typescript
// Step 1: Imagine a third-party library defines options.
interface LibraryOptions {
  apiKey: string;
  timeout?: number;
  retries?: number;
}

// Step 2: Extend with application-specific options.
type MyLibraryOptions = LibraryOptions & {
  customLogger: (message: string) => void;
  environment: "development" | "production";
};

// Step 3: Create the extended options.
const options: MyLibraryOptions = {
  apiKey: "key-123",
  timeout: 5000,
  customLogger: (msg) => console.log(`[MY-APP] ${msg}`),
  environment: "production",
};

// Step 4: Use the extended options.
options.customLogger(`Environment: ${options.environment}`);
console.log(`API Key: ${options.apiKey}`);
console.log(`Timeout: ${options.timeout}`);

// Step 5: The extended type is assignable to the library options.
function libraryInit(libOptions: LibraryOptions): void {
  console.log(`Initializing library with key ${libOptions.apiKey}`);
}

libraryInit(options);  // ✅ Works — MyLibraryOptions has all LibraryOptions members

// Step 6: Generic extension utility.
type WithLogging<T> = T & { log: (msg: string) => void };
type LoggedLibraryOptions = WithLogging<LibraryOptions>;

const loggedOptions: LoggedLibraryOptions = {
  apiKey: "key-456",
  log: (msg) => console.log(msg),
};

loggedOptions.log("Logged message");
```

**Expected Output:**
```
[MY-APP] Environment: production
API Key: key-123
Timeout: 5000
Initializing library with key key-123
Logged message
```

**Why This Output Occurs:** The `MyLibraryOptions` intersection adds `customLogger` and `environment` to `LibraryOptions`. The extended type is assignable to `LibraryOptions` because it has all required members. The library's `libraryInit` function accepts `LibraryOptions`, and `MyLibraryOptions` is structurally compatible. The generic `WithLogging<T>` utility demonstrates a reusable extension pattern.

### Real-World Cases

**Case 1: Passport.js Strategy Options**
Passport strategies use intersections to combine base strategy options with strategy-specific options.

**Case 2: Express Request Augmentation**
Express middleware augments `Request` with custom properties via intersections: `type AuthenticatedRequest = Request & { user: User }`.

**Case 3: Redux Store State Composition**
Redux store state composes from slice reducers: `type RootState = UserState & OrderState & ProductState`.

**Case 4: Form Library Option Extension**
Form libraries (Formik, React Hook Form) use intersections to extend base form options with validation and submission options.

---

## References

- TypeScript Handbook: Intersection Types — https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types
- TypeScript Handbook: Unions and Intersection Types — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types
- TypeScript Handbook: Mixins — https://www.typescriptlang.org/docs/handbook/mixins.html
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript 2.2 Release Notes (Object Type) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-2.html
- TypeScript Deep Dive: Intersection Types — https://basarat.gitbook.io/typescript/type-system/intersection
- Effective TypeScript: Item 13 — Know the Differences Between type and interface
- Effective TypeScript: Item 14 — Use Type Operations and Generics to Avoid Repeating Yourself
- MDN: Spread Syntax — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax
- TypeScript Playground: Intersection Types — https://www.typescriptlang.org/play/typescript/primitives/intersection-types.ts.html
- Total TypeScript: Intersection Types — https://www.totaltypescript.com/intersection-types
- TypeScript ESLint: consistent-type-definitions — https://typescript-eslint.io/rules/consistent-type-definitions/