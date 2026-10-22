## Generic Interfaces

**Topic Overview.** A **generic interface** is a TypeScript interface that declares one or more **type parameters**, allowing its members to reference those parameters. It is the interface-level analogue of generic functions and classes: a single interface declaration describes a **family of object types** parameterized over one or more type variables, and callers supply **type arguments** to produce a concrete object type. The TypeScript Language Specification defines it precisely: “An interface with type parameters is called a generic interface. The type parameters of a generic interface declaration are in scope in the entire declaration and may be referenced in the InterfaceExtendsClause and ObjectType body”. Generic interfaces are purely compile-time constructs — they have **no runtime representation** and are **erased** during emission. They are the backbone of container types (`Box<T>`, `Repository<T>`), API response envelopes (`ApiResponse<T>`), and React prop contracts (`ListProps<T>`).

---

## 1. Generic Interfaces (General)

### Definitions

**Core Definition.** A **generic interface** is an interface declaration that includes a **type parameter list** in angle brackets after the interface name, introducing one or more type variables that may be used throughout the interface body.

**Technical Definition.** An interface declaration has the grammatical form `interface BindingIdentifier TypeParameters? InterfaceExtendsClause? ObjectType`. When `TypeParameters` is present, the interface is generic. The type parameters are **in scope in the entire declaration**, including the `extends` clause and the object-type body. A generic interface is a **type constructor**: it is not itself a type until supplied with type arguments (unless all type parameters have defaults). Generic interfaces participate in TypeScript’s **structural type system**: an object type with the required members is compatible with a generic interface instantiation, regardless of how it was declared.

**Beginner-Friendly Explanation.** A generic interface is **a blueprint for objects that can hold or produce different types**. Instead of writing separate interfaces for a box of numbers and a box of strings, you write `Box<T>` once, and then use `Box<number>` or `Box<string>` as needed. The `<T>` is a placeholder that gets replaced with a real type when you use the interface.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Parameterized** | Declares one or more type parameters that are placeholders for concrete types. |
| **Compile-time only** | Erased at runtime; no runtime representation or reflection. |
| **Structural** | Compatibility is determined by members, not by nominal identity. |
| **Extensible** | May extend other interfaces or classes, and the `extends` clause may reference type parameters. |
| **Mergeable** | Multiple declarations of the same generic interface must have **identical type parameter lists**; declaration merging applies. |
| **Constraint-supporting** | Type parameters may be constrained with `extends` and defaulted with `=`. |

### Prerequisites

- **TypeScript fundamentals**: variables, functions, object types, and basic type annotations.
- **Basic generics**: understanding type parameters and type arguments (covered in Generic Functions).
- **Structural typing**: awareness that TypeScript compares object shapes by members, not by name.
- **A TypeScript compiler or editor** to run the examples.

### Related Programming Areas

- **Generic functions and generic classes**: the same type-parameter mechanics apply across all three constructs.
- **Utility types**: `Partial<T>`, `Readonly<T>`, and `Record<K, V>` are built on generic type mappings.
- **Object-oriented design**: generic interfaces model repositories, containers, and service contracts.
- **React and component libraries**: generic props interfaces type components that render arbitrary data.

### Core Concepts / Features

The following sections cover **generic interfaces** under four headings: **Generic Interface Declaration**, **Generic Interface Usage**, and **Generic Interface Best Practices**. Each follows the uniform structure: definitions, purposes (each beginning with “To”), syntax rules, annotated executable examples with outputs, and real-world cases.

---

## 2. Generic Interface Declaration

### Definitions

**Core Definition.** A **generic interface declaration** is the syntactic form that introduces a generic interface, consisting of the `interface` keyword, the interface name, a **type parameter list**, an optional `extends` clause, and the interface body.

**Technical Definition.** The declaration `interface Name<T, U extends Constraint = Default> extends Base<T> { ... }` introduces a named generic type in the containing declaration space. The type parameters are declared in angle brackets after the name and are in scope throughout the `extends` clause and the body. Type parameters may be **constrained** with `extends` and **defaulted** with `=`. When multiple declarations of the same interface exist, all must have **identical type parameter lists**; otherwise, declaration merging is an error.

**Beginner-Friendly Explanation.** A generic interface declaration is **where you write the blueprint**. You write `interface Box<T> { value: T }` to say “a `Box` holds a value of some type `T`.” The `<T>` introduces the placeholder, and `T` is used inside the body wherever the actual type should appear.

### Purposes

- **To introduce a reusable, parameterized object type** that can be specialized with different type arguments.
- **To relate multiple members** of an interface through shared type parameters, such as a key type and a value type.
- **To constrain type parameters** so the interface can safely reference members of the constraint.
- **To provide defaults** so callers can omit type arguments when a sensible default exists.
- **To extend other generic interfaces**, composing type-level abstractions.

### Syntax Rules and Structure

**Complete general syntax:**

```typescript
interface InterfaceName<TypeParam, OtherParam extends Constraint = Default>
  extends BaseInterface<TypeParam> {
  property: TypeParam;
  method(arg: OtherParam): TypeParam;
}
```

**Component breakdown:**

- `interface` — the declaration keyword.
- `InterfaceName` — the interface identifier.
- `<TypeParam, OtherParam extends Constraint = Default>` — the **type parameter list**:
  - `TypeParam` — an unconstrained type parameter.
  - `OtherParam extends Constraint` — a constrained type parameter; inside the body, values of this type may use members of `Constraint`.
  - `= Default` — a default type used when no type argument is supplied.
- `extends BaseInterface<TypeParam>` — an optional `extends` clause that may reference type parameters.
- `{ property: TypeParam; method(...): TypeParam; }` — the interface body, which may reference the type parameters.

**Syntax rules.**

- Type parameters are declared in angle brackets after the interface name.
- The type parameters are in scope in the entire declaration, including the `extends` clause and the body.
- Required type parameters must **not** follow defaulted type parameters; `<T = string, U>` is a compile-time error.
- When an interface has multiple declarations, all declarations must have **identical type parameter lists** for merging to succeed.
- Type parameters may be documented with the TSDoc `@typeParam` tag.

**Constraints and limitations.**

- Generic interfaces have **no runtime representation**; they cannot be used with `instanceof` or runtime reflection.
- An interface cannot directly or indirectly extend itself, regardless of type arguments.
- Over-constraining a type parameter couples the interface to a specific implementation and limits reuse; constrain to the **minimal interface** needed.
- Defaults should be **meaningful**; a default that is not useful without specialization creates a false sense of optionality.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic generic interface declaration

```typescript
// example_generic_interface_declaration.ts
// Step 1: Declare a generic interface with one type parameter.
interface Box<T> {
  value: T;               // property typed by T
  getValue(): T;          // method returning T
}

// Step 2: Use the interface with a concrete type argument.
const numberBox: Box<number> = {
  value: 42,
  getValue() {
    return this.value;
  },
};

// Step 3: Use with another type argument.
const stringBox: Box<string> = {
  value: "hello",
  getValue() {
    return this.value;
  },
};

console.log(numberBox.getValue());   // Expected: 42
console.log(stringBox.getValue());   // Expected: hello
```

**Expected Output:**
```
42
hello
```

**Why this output occurs.** `Box<T>` declares a single type parameter `T` and uses it in both the `value` property and the `getValue` return type. `Box<number>` requires `value: number` and `getValue(): number`; `Box<string>` requires the string equivalents. The compiler checks each object against its instantiated interface, then erases the interface entirely during emission.

#### Example 2: Constrained and defaulted type parameters

```typescript
// example_generic_interface_constraint_default.ts
// Step 1: Declare an interface with a constrained and defaulted parameter.
interface Repository<T extends { id: string } = { id: string }> {
  findById(id: string): T | undefined;
  save(entity: T): void;
}

// Step 2: Use the default type argument (omit <...>).
const defaultRepo: Repository = {
  findById: (id) => ({ id }),          // T defaults to { id: string }
  save: (entity) => console.log(entity),
};

// Step 3: Use an explicit type argument that satisfies the constraint.
interface User {
  id: string;
  name: string;
}

const userRepo: Repository<User> = {
  findById: (id) => ({ id, name: "Ada" }),
  save: (user) => console.log(user.name),
};

console.log(defaultRepo.findById("1")?.id);   // Expected: 1
userRepo.save({ id: "2", name: "Grace" });      // Expected: Grace
```

**Expected Output:**
```
1
Grace
```

**Why this output occurs.** The constraint `T extends { id: string }` guarantees that every `T` has an `id` property, making `findById`’s usage safe. The default `= { id: string }` provides a fallback when no type argument is supplied. `User` satisfies the constraint and is used explicitly. This illustrates both constraint and default syntax in a single declaration.

#### Example 3: Generic interface extending another generic interface

```typescript
// example_generic_interface_extends.ts
interface Container<T> {
  value: T;
}

// Extends a generic interface and preserves the type parameter.
interface NamedContainer<T> extends Container<T> {
  name: string;
}

const named: NamedContainer<number> = {
  name: "answer",
  value: 42,
};

console.log(named.name, named.value);   // Expected: answer 42
```

**Expected Output:**
```
answer 42
```

**Why this output occurs.** `NamedContainer<T>` extends `Container<T>`, inheriting the `value: T` member and adding `name: string`. The type parameter `T` is in scope in the `extends` clause, allowing the parent to be instantiated with the same type.

### Real-World Cases

- **API response envelopes**: `ApiResponse<T> { data: T; status: number }` describes typed responses without duplicating the envelope for each payload type.
- **Repositories**: `Repository<T extends Entity>` defines `findById`, `save`, and `findAll` operations for any entity type.
- **React props**: `ListProps<T> { items: T[]; renderItem: (item: T) => ReactNode }` types generic list components.

---

## 3. Generic Interface Usage

### Definitions

**Core Definition.** **Generic interface usage** refers to **referencing a generic interface** with type arguments — either explicitly in angle brackets or implicitly through defaults — to produce a concrete object type.

**Technical Definition.** A generic interface is used in a **type reference** such as `Box<number>` or, when all type parameters have defaults, `Box`. Type arguments are checked against the type parameter constraints. The position of the type parameter matters: `<T>` on the **interface name** makes the **whole interface** generic; `<T>` on a **call signature** makes **individual methods** generic. This distinction determines whether the type is fixed for the entire object or per invocation.

**Beginner-Friendly Explanation.** Using a generic interface means **filling in the placeholder**. You write `Box<number>` to say “a box of numbers.” If the interface has a default, you can omit the angle brackets. The type argument you supply is checked against any constraint the interface declares.

### Purposes

- **To instantiate a generic interface** with concrete types, producing a usable object type.
- **To fix a type for the entire object** when the type parameter is on the interface name.
- **To allow per-method type flexibility** when the type parameter is on a call signature.
- **To omit type arguments** when defaults provide a sensible fallback.

### Syntax Rules and Structure

**General syntax (explicit type argument):**

```typescript
const value: InterfaceName<TypeArgument> = { ... };
```

**Component breakdown:**

- `InterfaceName<TypeArgument>` — a type reference to the generic interface with an explicit type argument.
- `TypeArgument` — any type that satisfies the type parameter’s constraint (if present).

**Syntax rules.**

- Type arguments are supplied in angle brackets after the interface name.
- When the interface name has type parameters, the **whole interface** is generic; the type argument fixes the type for all members.
- When a **call signature** has type parameters, the **function** is generic; each call may use a different type argument.
- If all type parameters have defaults, the angle brackets may be omitted: `const x: Interface = ...` uses the defaults.

**Constraints and limitations.**

- Supplying a type argument that does not satisfy the constraint is a compile-time error.
- Supplying the wrong number of type arguments is an error unless the remaining parameters have defaults.
- Type arguments are erased at runtime; they have no runtime representation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit type argument usage

```typescript
// example_generic_interface_usage.ts
interface KeyValuePair<K, V> {
  key: K;
  value: V;
}

// Step 1: Supply both type arguments explicitly.
const pair: KeyValuePair<string, number> = {
  key: "age",
  value: 30,
};

console.log(pair.key, pair.value);   // Expected: age 30
```

**Expected Output:**
```
age 30
```

**Why this output occurs.** `KeyValuePair<string, number>` fixes `K = string` and `V = number`. The object must have a string `key` and a numeric `value`. The compiler checks this before erasure.

#### Example 2: Interface-level type parameter vs call-signature type parameter

```typescript
// example_interface_vs_call_signature.ts
// Interface-level type parameter: the type is fixed for the object.
interface FixedIdentity<T> {
  (arg: T): T;
}

// Call-signature type parameter: each call may use a different type.
interface GenericIdentity {
  <T>(arg: T): T;
}

const fixed: FixedIdentity<number> = (arg) => arg;
const generic: GenericIdentity = (arg) => arg;

console.log(fixed(1));         // Expected: 1
console.log(generic("a"));     // Expected: a
console.log(generic(2));       // Expected: 2
```

**Expected Output:**
```
1
a
2
```

**Why this output occurs.** `FixedIdentity<number>` fixes `T = number` for the entire function type, so only numbers are accepted. `GenericIdentity` places `<T>` on the call signature, so each invocation infers its own `T` — strings and numbers are both accepted. This distinction is central to designing generic interfaces.

#### Example 3: Using defaults to omit type arguments

```typescript
// example_interface_usage_default.ts
interface Settings<T = string> {
  value: T;
}

const defaultSettings: Settings = { value: "dark" };   // T defaults to string
const numberSettings: Settings<number> = { value: 1 }; // explicit override

console.log(defaultSettings.value);   // Expected: dark
console.log(numberSettings.value);    // Expected: 1
```

**Expected Output:**
```
dark
1
```

**Why this output occurs.** `Settings` declares `T = string`. Omitting the type argument uses the default; supplying `Settings<number>` overrides it.

### Real-World Cases

- **API responses**: `ApiResponse<User>` fixes the payload type for the entire response object.
- **Generic call signatures**: `Comparator<T>` interfaces with a call signature allow each comparison to infer its own type.
- **Defaulted settings**: `Options<T = {}>` permits simple call sites to omit the type argument while still allowing specialization.

---

## 4. Generic Interface Best Practices

### Definitions

**Core Definition.** **Generic interface best practices** are guidelines for declaring and using generic interfaces in ways that maximize **reusability, type safety, inference success, and clarity**, avoiding over-constraining, unnecessary defaults, and ambiguous type-parameter placement.

**Technical Definition.** Best practices include: placing type parameters on the **interface name** when the type should be fixed for the whole object, and on **call signatures** when per-invocation flexibility is desired; constraining to the **minimal interface** actually used; providing **meaningful defaults** only when the type is genuinely usable without specialization; and ensuring type parameters appear in **two or more member positions** to establish relationships. Over-constraining couples the interface to specific implementations and limits reuse, violating the Interface Segregation Principle.

**Beginner-Friendly Explanation.** Best practices help you write generic interfaces that are **actually useful and not confusing**. Use a type parameter only when it connects multiple members. Put the type parameter where it makes sense — on the interface if the type is fixed, on a method if each call can differ. Constrain only what you need, and avoid defaults that are not meaningful.

### Purposes

- **To maximize inference success** by placing type parameters where they can be inferred from usage.
- **To preserve genuine type relationships** between members of the interface.
- **To minimize coupling** by constraining to the smallest interface that satisfies the implementation’s needs.
- **To avoid misleading APIs** created by unnecessary defaults or over-constraining.
- **To keep interfaces readable and maintainable** by using descriptive type parameter names and documenting their meaning.

### Syntax Rules and Structure

**Key best-practice rules:**

1. **Place type parameters deliberately.** Use `interface Foo<T> { ... }` when `T` is fixed for the object; use `interface Foo { <T>(arg: T): T }` when each call may use a different type.
2. **Constrain to the minimal interface.** Use the smallest set of members the interface actually accesses, not the full type.
3. **Avoid unnecessary defaults.** Provide a default only when the type is genuinely usable without specialization; otherwise, omit it.
4. **Ensure type parameters appear in at least two positions.** A type parameter used only once relates nothing and is likely unnecessary.
5. **Use descriptive type parameter names** (`TKey`, `TValue`) when single-letter names are ambiguous.
6. **Document type parameters** with TSDoc `@typeParam`.
7. **Prefer interfaces for object shapes**; use type aliases for unions and intersections.

**Constraints and limitations.**

- Over-constraining reduces reusability and couples the interface to a specific implementation.
- Defaults that are not meaningful create a false sense of flexibility.
- Interface-level type parameters that are never used in the body are a code smell.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface-level vs call-signature type parameter

```typescript
// example_best_placement.ts
// GOOD: T is fixed for the whole object.
interface Parser<T> {
  parse(input: string): T;
}

const numberParser: Parser<number> = {
  parse: (input) => Number(input),
};

// GOOD: each call may use a different T.
interface ParseFn {
  <T>(input: string): T;
}

const parse: ParseFn = (input) => JSON.parse(input) as any;

console.log(numberParser.parse("42"));   // Expected: 42
console.log(parse<{ name: string }>('{"name":"Ada"}').name); // Expected: Ada
```

**Expected Output:**
```
42
Ada
```

**Why this output occurs.** `Parser<number>` fixes `T = number` for the entire parser object. `ParseFn` places `<T>` on the call signature, so each invocation supplies its own type argument. The placement matches the intended semantics.

#### Example 2: Minimal constraint

```typescript
// example_best_minimal_constraint.ts
// GOOD: constraint includes only what the interface uses.
interface HasId {
  id: string;
}

interface Repository<T extends HasId> {
  findById(id: string): T | undefined;
}

// Over-constrained: requires full User even when only id is needed.
interface OverConstrainedRepository<T extends User> {
  findById(id: string): T | undefined;
}

interface User {
  id: string;
  name: string;
  email: string;
}

const repo: Repository<{ id: string; name: string }> = {
  findById: (id) => ({ id, name: "Ada" }),
};

console.log(repo.findById("1")?.name);   // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `HasId` includes only the `id` property that `findById` actually uses. The `Repository` constraint is satisfied by any type with an `id`, including the minimal `{ id: string; name: string }`. The over-constrained version would require `email` unnecessarily, limiting reuse.

#### Example 3: Meaningful defaults only

```typescript
// example_best_defaults.ts
// GOOD: string is a meaningful default for a formatter.
interface Formatter<T = string> {
  format(value: T): string;
}

// BAD: never is not meaningful without specialization.
// interface Repository<T = never> { ... }

const stringFormatter: Formatter = {
  format: (v) => v.toUpperCase(),
};

console.log(stringFormatter.format("hello"));   // Expected: HELLO
```

**Expected Output:**
```
HELLO
```

**Why this output occurs.** `T = string` is meaningful because a string formatter is a valid, useful configuration without specialization. A default like `never` would mislead callers into thinking the type is optional when the interface is not useful without it.

### Real-World Cases

- **Library APIs**: Minimal constraints and deliberate type-parameter placement produce ergonomic, inferable interfaces.
- **React props**: `ListProps<T>` places `T` on the interface name because the component renders a list of a fixed element type.
- **Comparator APIs**: `Comparator` with a call signature `<T>(a: T, b: T) => number` allows each comparison to infer its own type.

---

## References

- TypeScript Language Specification — Interfaces (Generic Interfaces) - https://chromium.googlesource.com/external/github.com/kythe/kythe/+/7c2c91248ad9bc8d22dfaa00764c6381af512113/third_party/typescript/doc/spec.md#12
- TypeScript Handbook — Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook — Interfaces - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Generics: A Complete Guide (Syncfusion) - https://www.syncfusion.com/blogs/post/typescript-generics-guide
- Generic Default — Educative Course Material - https://github.com/apple-wp-xmr/educative.io_courses/blob/61e71aeee243445ef1d68bab52935243bc211265/Learn%20TypeScript_%20The%20Complete%20Course%20for%20Beginners%20-%20Learn%20Interactively/35_Generic_Default.pdf
- Generic call signatures and type inference — Stack Overflow - https://stackoverflow.com/questions/65158607
- Constrain, Don’t Over-Constrain — Generic Patterns - https://github.com/pproenca/dot-skills/blob/cf93c57cac89d6fc3e4194686000411567f5caf3/skills/.experimental/typescript-refactor/references/generic-constrain-dont-overconstrain.md
- TypeScript and Architecture Patterns — Generic Parameter Defaults - https://raw.githubusercontent.com/aiskillstore/marketplace/9575dcdb5d4e987a683699070eb27ebfe76336b2/marketplace/skills/barnhardt-enterprises-inc/typescript-strict-guard/generic-patterns.md