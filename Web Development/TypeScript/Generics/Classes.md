## Generic Classes

**Topic Overview.** A **generic class** is a TypeScript class that declares one or more **type parameters** — type-level variables that stand in for concrete types supplied when the class is instantiated. Unlike a non-generic class, which fixes its member types at declaration time, a generic class is a **type constructor**: `Box<T>` describes a family of classes, and `new Box<number>()` produces a concrete instance whose members are specialized to `number`. Generic classes are the class-level expression of TypeScript’s reusability model: they allow a single implementation of a data structure, repository, or container to work over arbitrary types while **preserving type relationships** between constructor arguments, properties, and method signatures. A critical and often misunderstood constraint is that generic classes are **generic only over their instance side**, not their static side: **static members cannot reference the class’s type parameter** . Generic classes are purely compile-time constructs; all type parameters are **erased** during emission and have no runtime representation.

---

## 1. Generic Classes (General)

### Definitions

**Core Definition.** A **generic class** is a class declaration that includes a **type parameter list** in angle brackets after the class name, introducing one or more type variables that may be used in instance properties, constructor parameters, method signatures, and the class heritage clause.

**Technical Definition.** The TypeScript Language Specification defines it precisely: “A class may optionally have type parameters that serve as placeholders for … is referenced in type references. A class with type parameters is called a generic class. The type parameters of a generic class declaration are in scope in the entire declaration and may be referenced in the ClassHeritage and ClassBody” . A generic class has two sides to its type: the **static side** (the constructor function) and the **instance side** (the instances produced by the constructor). The class’s type parameters are in scope only on the **instance side**; static members cannot use the class’s type parameter . Generic classes participate in TypeScript’s structural type system and are erased at runtime.

**Beginner-Friendly Explanation.** A generic class is **a class with a placeholder type**. You write `class Box<T> { value: T }` to say “a `Box` holds a value of some type `T`.” When you write `new Box<number>()`, `T` becomes `number` for that instance, and the `value` property must be a number. The class works for any type, but each instance tracks its type consistently.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Instance-side only** | Type parameters apply to instance members, not static members . |
| **Type constructor** | `Box<T>` is not a usable type until supplied with type arguments (unless defaults are present). |
| **Compile-time only** | Erased at runtime; no type information survives emission. |
| **Structural compatibility** | An object is compatible with `Box<number>` if it has the required shape, regardless of how it was created. |
| **Constraint-capable** | Type parameters may be constrained with `extends` and defaulted with `=`. |
| **Heritage-capable** | Type parameters may be used in `extends` and `implements` clauses. |

### Prerequisites

- **TypeScript fundamentals**: classes, constructors, properties, methods, access modifiers.
- **Basic generics**: type parameters and type arguments (covered in Generic Functions).
- **Structural typing**: awareness that TypeScript compares object shapes by members, not by name.
- **A TypeScript compiler or editor** to run the examples.

### Related Programming Areas

- **Generic functions and generic interfaces**: the same type-parameter mechanics apply across constructs.
- **Data structures**: stacks, queues, linked lists, and trees are natural generic class candidates.
- **Repository and service patterns**: generic repositories abstract CRUD operations over entity types.
- **Design patterns**: generic factories, builders, and mixins rely on class-level type parameters .

---

## 2. Generic Class Declaration

### Definitions

**Core Definition.** A **generic class declaration** is the syntactic form that introduces a generic class, consisting of the `class` keyword, the class name, a **type parameter list**, an optional heritage clause, and the class body.

**Technical Definition.** The declaration `class Name<T, U extends Constraint = Default> extends Base<T> implements Interface<U> { ... }` introduces a named generic class. The type parameters are declared in angle brackets after the class name and are in scope throughout the **instance side** of the class — property declarations, constructor parameters, method signatures, and the heritage clause. They are **not** in scope in static members . Type parameters may be constrained with `extends` and defaulted with `=`. A later type parameter may reference an earlier one in its constraint (e.g., `<T, K extends keyof T>`).

**Beginner-Friendly Explanation.** A generic class declaration is **where you write the blueprint**. You write `class Stack<T> { private items: T[] }` to say “a `Stack` holds items of some type `T`.” The `<T>` introduces the placeholder, and `T` is used throughout the instance members wherever the actual type should appear.

### Purposes

- **To introduce a reusable, parameterized class** that can be instantiated with different type arguments.
- **To enforce type consistency across instance properties and methods**, so all members work with the same type.
- **To constrain type parameters** so the class can safely access members of the constraint.
- **To provide defaults** so callers can omit type arguments when a sensible fallback exists.
- **To compose with inheritance and interfaces** through generic heritage clauses.

### Syntax Rules and Structure

**Complete general syntax:**

```typescript
class ClassName<TypeParam, OtherParam extends Constraint = Default>
  extends BaseClass<TypeParam>
  implements SomeInterface<OtherParam> {
  private property: TypeParam;
  constructor(public param: TypeParam) { ... }
  method(arg: OtherParam): TypeParam { ... }
  static staticMethod(): void { /* cannot reference TypeParam */ }
}
```

**Component breakdown:**

- `class` — the declaration keyword.
- `ClassName` — the class identifier.
- `<TypeParam, OtherParam extends Constraint = Default>` — the **type parameter list**:
  - `TypeParam` — an unconstrained type parameter.
  - `OtherParam extends Constraint` — a constrained type parameter; inside instance members, values of this type may use members of `Constraint`.
  - `= Default` — a fallback type used when no type argument is supplied.
- `extends BaseClass<TypeParam>` — an optional heritage clause that may reference type parameters.
- `implements SomeInterface<OtherParam>` — an optional interface implementation clause.
- Instance members — properties, constructor parameters, and methods that may use the type parameters.
- `static staticMethod` — static members **cannot** reference the class’s type parameters .

**Syntax rules.**

- Type parameters are declared in angle brackets after the class name.
- Type parameters are in scope in the **entire declaration** except static members .
- Required type parameters must **not** follow defaulted type parameters.
- Constraints may reference other type parameters (e.g., `<T, K extends keyof T>`).
- Type parameters may be documented with TSDoc `@typeParam`.

**Constraints and limitations.**

- **Static members cannot use class type parameters**; this is a fundamental rule, not a limitation that can be worked around with syntax .
- Generic classes have **no runtime representation**; they cannot be used with `instanceof` in a way that inspects type arguments.
- Over-constraining couples the class to a specific implementation and limits reuse.
- Defaults that are not meaningful create a false sense of optionality.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic generic class declaration

```typescript
// example_generic_class_declaration.ts
// Step 1: Declare a generic class with one type parameter.
class Box<T> {
  private content: T;

  constructor(value: T) {
    this.content = value;   // T is in scope on the instance side
  }

  getValue(): T {
    return this.content;
  }
}

// Step 2: Instantiate with number.
const numberBox = new Box<number>(42);
// Step 3: Instantiate with string.
const stringBox = new Box<string>("hello");

console.log(numberBox.getValue());   // Expected: 42
console.log(stringBox.getValue());   // Expected: hello
```

**Expected Output:**
```
42
hello
```

**Why this output occurs.** `Box<T>` declares a single type parameter and uses it in the private `content` property, the constructor parameter, and the `getValue` return type. `new Box<number>(42)` fixes `T = number` for that instance; `new Box<string>("hello")` fixes `T = string`. The compiler checks each instantiation, then erases the type parameter entirely.

#### Example 2: Constrained and defaulted type parameters

```typescript
// example_generic_class_constraint_default.ts
interface Identifiable {
  id: string;
}

class Repository<T extends Identifiable = Identifiable> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  findById(id: string): T | undefined {
    return this.items.find((item) => item.id === id);
  }
}

// Default type argument.
const defaultRepo = new Repository();
defaultRepo.add({ id: "1" });
console.log(defaultRepo.findById("1")?.id);   // Expected: 1

// Explicit type argument satisfying the constraint.
interface User extends Identifiable {
  name: string;
}
const userRepo = new Repository<User>();
userRepo.add({ id: "2", name: "Ada" });
console.log(userRepo.findById("2")?.name);     // Expected: Ada
```

**Expected Output:**
```
1
Ada
```

**Why this output occurs.** The constraint `T extends Identifiable` guarantees that every `T` has an `id: string` property, making `findById`’s comparison legal. The default `= Identifiable` provides a fallback when no type argument is supplied. `User` satisfies the constraint and is used explicitly.

#### Example 3: Static members cannot use the class type parameter

```typescript
// example_generic_class_static.ts
class Box<T> {
  static defaultValue: T;   // Error: Static members cannot reference class type parameters.
}
```

**Expected Output (compile-time error):**
```
Static members cannot reference class type parameters.
```

**Why this output occurs.** A class’s type parameters are in scope only on the **instance side**. The static side — the constructor function — exists once per class, regardless of how many type arguments are used. Allowing `static defaultValue: T` would require a separate static property for every possible `T`, which is not how JavaScript constructors work. This is a deliberate language rule .

### Real-World Cases

- **Data structures**: `Stack<T>`, `Queue<T>`, and `LinkedList<T>` are generic classes that enforce element-type consistency.
- **Repositories**: `Repository<T extends Entity>` provides CRUD operations for any entity type without duplication.
- **Containers and wrappers**: `Box<T>`, `Result<T>`, and `Cache<K, V>` model typed containers.

---

## 3. Generic Class Usage

### Definitions

**Core Definition.** **Generic class usage** refers to **instantiating or referencing** a generic class with type arguments — either explicitly in angle brackets or implicitly through inference from constructor arguments — to produce a concrete instance or type.

**Technical Definition.** A generic class is used in a **`new` expression** (`new Box<number>(42)`) or a **type reference** (`Box<number>`). Type arguments are checked against the type parameter constraints. When type arguments are omitted, TypeScript performs **type argument inference** from the constructor arguments; if a type parameter appears in a constructor parameter, its type argument may be inferred from the supplied value. Type arguments may be supplied on either the left-hand side (as a type annotation) or the right-hand side (as part of the constructor call), and consistency rules apply.

**Beginner-Friendly Explanation.** Using a generic class means **creating an instance or referring to its type**. You write `new Box<number>(42)` to create a box of numbers, or `const b: Box<string> = ...` to declare a variable of that type. TypeScript can often infer the type argument from the constructor argument, so `new Box(42)` also works.

### Purposes

- **To instantiate a generic class** with concrete types, producing a usable object.
- **To fix the type argument for an instance**, ensuring all members work with that type.
- **To rely on inference** from constructor arguments for concise instantiation.
- **To reference the generic class as a type** in annotations, parameters, and return types.

### Syntax Rules and Structure

**General syntax (explicit instantiation):**

```typescript
const instance = new ClassName<TypeArgument>(constructorArgs);
const typed: ClassName<TypeArgument> = instance;
```

**Component breakdown:**

- `new ClassName<TypeArgument>` — explicit type argument on the constructor call.
- `ClassName<TypeArgument>` — a type reference to the instantiated class.
- `constructorArgs` — arguments checked against the constructor’s parameter types after substituting the type argument.

**Syntax rules.**

- Type arguments are supplied in angle brackets after the class name in the `new` expression or type reference.
- When type arguments are omitted, inference uses the constructor arguments; if no constructor parameter uses the type parameter, the type argument defaults to `unknown` or a default if provided.
- Type arguments may be supplied on the left-hand side or the right-hand side of an assignment, but consistency rules apply; ESLint’s `consistent-generic-constructors` rule enforces one convention .

**Constraints and limitations.**

- Supplying a type argument that does not satisfy the constraint is a compile-time error.
- Inference from constructor arguments may produce a wider type than intended; explicit type arguments resolve ambiguity.
- Type arguments are erased at runtime; `new Box<number>(42) instanceof Box` is `true`, but the type argument is not recoverable.

### Annotated Complete Step by Step Code Examples

#### Example 1: Explicit and inferred instantiation

```typescript
// example_generic_class_usage.ts
class KeyValuePair<K, V> {
  constructor(public key: K, public value: V) {}
}

// Explicit type arguments.
const pair1 = new KeyValuePair<number, string>(1, "One");

// Inferred type arguments.
const pair2 = new KeyValuePair("isTrue", true);

console.log(pair1.key, pair1.value);   // Expected: 1 One
console.log(pair2.key, pair2.value);   // Expected: isTrue true
```

**Expected Output:**
```
1 One
isTrue true
```

**Why this output occurs.** `new KeyValuePair<number, string>(1, "One")` explicitly sets `K = number` and `V = string`. `new KeyValuePair("isTrue", true)` infers `K = string` and `V = boolean` from the constructor arguments. Both produce correctly typed pairs.

#### Example 2: Type reference usage

```typescript
// example_generic_class_usage_type.ts
class Stack<T> {
  private items: T[] = [];
  push(item: T): void { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
}

// Type reference in a variable annotation.
const numberStack: Stack<number> = new Stack<number>();
numberStack.push(1);
console.log(numberStack.pop());   // Expected: 1

// Type reference in a function parameter.
function drain<T>(stack: Stack<T>): T[] {
  const result: T[] = [];
  let item = stack.pop();
  while (item !== undefined) {
    result.push(item);
    item = stack.pop();
  }
  return result;
}

console.log(drain(numberStack));   // Expected: []
```

**Expected Output:**
```
1
[]
```

**Why this output occurs.** `Stack<number>` is used both as a constructor instantiation and as a type annotation. `drain` accepts any `Stack<T>` and returns `T[]`, preserving the element type. The second call returns an empty array because the stack was already drained by `pop()`.

### Real-World Cases

- **Collections**: `new Map<string, number>()` supplies key and value type arguments.
- **Repositories**: `new Repository<User>()` fixes the entity type for a repository instance.
- **React state**: Class components may use generic props and state types.

---

## 4. Generic Class Best Practices

### Definitions

**Core Definition.** **Generic class best practices** are guidelines for declaring and using generic classes in ways that maximize **reusability, type safety, inference success, and clarity**, while respecting the instance-side-only rule and avoiding unnecessary type parameters, over-constraining, and misleading defaults.

**Technical Definition.** Best practices include: using generic classes only when **at least two concrete instantiations share the structure**; respecting the **instance-side-only scope** of type parameters; constraining to the **minimal interface** the class actually uses; providing **meaningful defaults** only when the class is usable without specialization; and ensuring type parameters appear in **two or more member positions** to establish relationships. The TypeScript Handbook’s guidance for generic functions — “push type parameters down” and “use fewer type parameters” — applies analogously to classes.

**Beginner-Friendly Explanation.** Best practices help you write generic classes that are **actually useful and not confusing**. Use a generic class only when you genuinely need it for multiple types. Remember that static members cannot use the class’s type parameter. Constrain only what you need, and avoid defaults that are not meaningful.

### Purposes

- **To justify the generic class’s existence** by ensuring it serves multiple concrete types.
- **To respect the static/instance split** and avoid impossible static type-parameter usage.
- **To minimize coupling** by constraining to the smallest interface that satisfies the class’s needs.
- **To maximize inference success** so callers rarely need explicit type arguments.
- **To keep the class readable and maintainable** with descriptive type parameter names and documentation.

### Syntax Rules and Structure

**Key best-practice rules:**

1. **Drop the type parameter when only one concrete type uses it.** Generics earn their cost when at least two concrete types share the structure. A class with one `<T>` and a single instantiation should be specialized to that concrete type until a second use appears .
2. **Respect the instance-side-only rule.** Static members cannot use the class’s type parameter; if static generic behavior is needed, use a generic static method with its own type parameter or a separate generic function .
3. **Constrain to the minimal interface.** Use the smallest set of members the class actually accesses, not the full type.
4. **Provide meaningful defaults only.** A default is appropriate only when the class is genuinely usable without specialization.
5. **Ensure type parameters appear in at least two member positions.** A type parameter used only once relates nothing and is likely unnecessary.
6. **Use descriptive type parameter names** (`TKey`, `TValue`) when single-letter names are ambiguous.
7. **Document type parameters** with TSDoc `@typeParam`.

**Constraints and limitations.**

- Over-constraining reduces reusability and couples the class to a specific implementation.
- Defaults that are not meaningful create a false sense of flexibility.
- A generic class with a single instantiation adds complexity without benefit.
- Static members cannot reference class type parameters under any syntax.

### Annotated Complete Step by Step Code Examples

#### Example 1: Drop unnecessary generic parameters

```typescript
// example_best_single_instantiation.ts
// BAD: only one concrete instantiation uses the type parameter.
class UserRepository<T> {
  private items: T[] = [];
  add(item: T): void { this.items.push(item); }
}
// Only ever used as UserRepository<User>; the generic adds no value.

// GOOD: specialize to the concrete type until a second use appears.
interface User {
  id: string;
  name: string;
}

class UserRepositoryConcrete {
  private items: User[] = [];
  add(item: User): void { this.items.push(item); }
}

const repo = new UserRepositoryConcrete();
repo.add({ id: "1", name: "Ada" });
console.log(repo);
// Expected: UserRepositoryConcrete { items: [ { id: '1', name: 'Ada' } ] }
```

**Expected Output:**
```
UserRepositoryConcrete { items: [ { id: '1', name: 'Ada' } ] }
```

**Why this output occurs.** The generic `UserRepository<T>` provides no benefit when only one concrete type is ever used. Specializing to `User` keeps the code simpler. When a second entity type (e.g., `Product`) appears, the generic can be reintroduced and justified by multiple instantiations .

#### Example 2: Static members with their own generic methods

```typescript
// example_best_static_generic.ts
class Factory {
  // GOOD: a static generic method declares its own type parameter.
  static create<T>(value: T): { value: T } {
    return { value };
  }
}

const a = Factory.create("hello");   // T = string
const b = Factory.create(42);        // T = number

console.log(a.value);   // Expected: hello
console.log(b.value);   // Expected: 42
```

**Expected Output:**
```
hello
42
```

**Why this output occurs.** The static method `create<T>` declares its **own** type parameter, separate from any class-level parameter. This is legal because the type parameter belongs to the method, not to the class. A class-level type parameter cannot be used in a static member, but a static generic method can declare its own .

#### Example 3: Minimal constraint

```typescript
// example_best_minimal_constraint.ts
// GOOD: constraint includes only what the class uses.
interface HasId {
  id: string;
}

class Repository<T extends HasId> {
  private items: T[] = [];
  add(item: T): void { this.items.push(item); }
  findById(id: string): T | undefined {
    return this.items.find((item) => item.id === id);
  }
}

// The minimal type { id: string; name: string } satisfies the constraint.
const repo = new Repository<{ id: string; name: string }>();
repo.add({ id: "1", name: "Ada" });
console.log(repo.findById("1")?.name);   // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `HasId` includes only the `id` property that `findById` actually uses. The constraint does not require unrelated members, so the repository is reusable for any type with an `id`. This is the class-level analogue of constraining to the minimal interface.

### Real-World Cases

- **Repository patterns**: Minimal constraints (`T extends HasId`) keep repositories reusable across entity types.
- **Static factories**: Generic static methods (`Factory.create<T>`) provide typed creation without violating the instance-side-only rule.
- **Library design**: Justifying generics by multiple concrete uses prevents over-engineering and keeps APIs simple.

---

## References

- TypeScript Handbook — Generics (Generic Classes) - https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-classes
- TypeScript Language Specification — Classes (Generic Classes) - https://github.com/microsoft/TypeScript/blob/main/doc/spec.md
- TypeScript Handbook — Generics (Static Members Cannot Reference Class Type Parameters) - https://github.com/Microsoft/TypeScript-Handbook/blob/master/pages/Generics.md
- TypeScript Playground — Generic Classes - https://www.typescriptlang.org/play/javascript/working-with-classes/generic-classes.ts.html
- Microsoft Learn — Define Generics in TypeScript (Generic Classes) - https://learn.microsoft.com/en-us/training/modules/typescript-generics/
- Pluralsight — Build Reusable Components with TypeScript Generics (Generic Repositories) - https://www.pluralsight.com/labs/codeLabs/guided-build-reusable-components-with-typescript-generics
- TypeScript ESLint — `consistent-generic-constructors` - https://typescript-eslint.io/rules/consistent-generic-constructors/
- TypeScript Handbook — Mixins (Generic Class Inheritance) - https://www.typescriptlang.org/docs/handbook/mixins.html
- Effective TypeScript — Avoid Unnecessary Type Parameters - https://github.com/danvk/effective-typescript