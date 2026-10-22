# Access Modifiers in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Access modifiers** (also called **visibility modifiers**) are TypeScript keywords that control the **compile-time accessibility** of class members — properties, methods, accessors, and constructors — across three contexts: the declaring class itself, its subclasses, and external code. TypeScript provides three access modifiers — `public`, `private`, and `protected` — plus the orthogonal `readonly` modifier, which restricts **assignment** rather than **access**. These modifiers are **erased** during compilation and have **no runtime effect**, with the important exception that ECMAScript `#` private fields (a distinct JavaScript feature) enforce privacy at runtime.

**Technical Definition.** Access modifiers are part of TypeScript’s **static checking layer** over JavaScript classes. The `public`, `private`, and `protected` modifiers establish a **visibility lattice**: `public` members are accessible from any location; `protected` members are accessible from the declaring class and its subclasses; `private` members are accessible only from within the declaring class. The `readonly` modifier is orthogonal — it prevents assignment outside the constructor while leaving the member’s visibility unchanged. These modifiers affect **structural compatibility**: classes with `private` or `protected` members are compatible only if the member originates from the **same declaration**, introducing nominal-like behavior into TypeScript’s otherwise structural system. All TypeScript-specific modifiers are erased; the emitted JavaScript contains ordinary properties, and `#` private fields are the only mechanism that provides **runtime-enforced** privacy.

**Beginner-Friendly Explanation.** Access modifiers are **labels that tell TypeScript who is allowed to touch a class member**. `public` means “anyone can use it.” `private` means “only this class can use it.” `protected` means “this class and its children can use it.” `readonly` means “set it once during construction, then leave it alone.” These labels help you catch mistakes early and document your intent. But remember: TypeScript’s labels are **only enforced by the compiler** — when the code runs as JavaScript, the labels are gone. If you need privacy that survives compilation, you must use JavaScript’s `#` private fields.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Compile-time enforcement** | `public`, `private`, `protected`, and `readonly` are checked by the TypeScript compiler only; they emit no runtime checks. |
| **Default visibility is public** | A member without a modifier is implicitly `public`; writing `public` is usually redundant. |
| **Three-level visibility lattice** | `private` (class only) ⊂ `protected` (class + subclasses) ⊂ `public` (everywhere). |
| **`readonly` is orthogonal** | `readonly` restricts assignment, not visibility; it can combine with any visibility modifier. |
| **Nominal-like compatibility** | `private` and `protected` members make classes compatible only when the member comes from the same declaration. |
| **Runtime privacy requires `#`** | ECMAScript private fields (`#field`) enforce privacy at runtime; TypeScript’s `private` does not. |
| **Cross-instance access** | TypeScript permits `private` access between instances of the **same class**, unlike some OOP languages. |
| **Erasure** | Modifiers disappear in emitted JavaScript; only `#` private fields have runtime representation. |

---

### Prerequisites

- **JavaScript classes**: `class`, `constructor`, fields, methods, and instantiation.
- **TypeScript class basics**: type annotations, constructors, and parameter properties.
- **Structural typing**: how TypeScript compares object shapes.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compile-time errors.

---

### Related Programming Areas

- **Object-oriented encapsulation** and information hiding.
- **Structural vs. nominal typing** in TypeScript.
- **ECMAScript private class fields** (`#field`) for runtime privacy.
- **Interface implementation** — interfaces cannot require `private` members from implementers.
- **Design principles**: encapsulation, the Principle of Least Privilege, and interface segregation.

---

### Core Concepts / Features

The following sections cover each requested modifier followed by best practices. Each follows the uniform structure.

---

## 1. `public`

### Definitions

**Core Definition.** **`public`** is the default access modifier in TypeScript. A `public` member is accessible from **anywhere**: inside the declaring class, from subclasses, and from external code.

**Technical Definition.** The `public` modifier imposes **no access restriction** during type checking. Because it is the default, explicitly writing `public` is redundant except in **parameter properties**, where it is required to create a field. Google’s TypeScript Style Guide explicitly states: “Never use the `public` modifier except when declaring non-readonly public parameter properties (in constructors)” [8†L5-L12]. Public members contribute to the **structural type** of the class and affect compatibility checks.

**Beginner-Friendly Explanation.** `public` means **“anyone can use this.”** It is the default, so you rarely need to write it. If a member has no modifier, it is public.

### Purposes

- **To serve as the default visibility** for class members.
- **To explicitly document** that a member is part of the public API, when team style permits.
- **To create public parameter properties** in constructors, where the keyword is syntactically required.

### Syntax Rules and Structure

**General syntax:**

```
public memberName: Type;
```

**Component breakdown:**

- `public` — visibility modifier.
- `memberName` — identifier.
- `: Type` — optional type annotation.

**Syntax rules.**

- `public` is the default; omitting it has the same effect.
- `public` can combine with `readonly` and `static`.
- In parameter properties, `public` creates a field and assigns the constructor argument.

**Constraints and limitations.**

- Explicit `public` is often considered redundant style; Google’s style guide recommends against it except for non-readonly parameter properties [8†L5-L12].
- Public members are part of the structural type and affect compatibility.

### Annotated Complete Step by Step Code Examples

#### Example 1: Public vs. default visibility

```typescript
// example_public.ts
// Step 1: Explicit public (redundant but legal).
class User {
  public name: string;

  // Step 2: Implicit public (default).
  email: string;

  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
  }
}

// Step 3: Both members are accessible externally.
const user = new User("Ada", "ada@example.com");
console.log(user.name);  // Expected: Ada
console.log(user.email); // Expected: ada@example.com
```

**Setup.** Compile with `npx tsc example_public.ts --strict`.

**Expected Output:**
```
Ada
ada@example.com
```

**Why this output occurs.** Both `name` (explicit `public`) and `email` (implicit `public`) are accessible from outside the class. The explicit `public` documents intent but does not change behavior.

---

#### Example 2: Public parameter property

```typescript
// example_public_param.ts
class Point {
  // public is required here to create the field.
  constructor(public x: number, public y: number) {}
}

const p = new Point(1, 2);
console.log(p.x, p.y); // Expected: 1 2
```

**Expected Output:**
```
1 2
```

**Why this output occurs.** The `public` parameter properties create public fields `x` and `y` and assign the constructor arguments. This is one of the few places where writing `public` is necessary and recommended.

---

### Real-World Cases

- **Public API classes** where members are intentionally exposed.
- **DTOs and parameter properties** where `public` is required for field creation.
- **Framework components** whose public methods are called by the framework.

---

## 2. `private`

### Definitions

**Core Definition.** **`private`** restricts member access to **within the declaring class only**. Subclasses and external code cannot access `private` members.

**Technical Definition.** The `private` modifier is enforced **at compile time** by TypeScript. It does not emit runtime access checks; the emitted JavaScript still allows access to the property. For **true runtime privacy**, ECMAScript `#` private fields must be used. Two classes with `private` members are **not structurally compatible** unless the private member originates from the **same declaration**, introducing nominal-like behavior. TypeScript allows **cross-instance `private` access**: a method of class `A` can access `private` members of **another instance of `A`** [16†L25-L31]. Private members are accessible via bracket notation during type checking, making them “soft private” and potentially easier to access for testing [16†L43-L46].

**Beginner-Friendly Explanation.** `private` means **“only this class can touch it.”** Other code — even subclasses — cannot read or write it. TypeScript will give you an error if you try. But this is a **compile-time** rule, not a runtime lock. If you need privacy that truly survives compilation, use `#privateField` instead.

### Purposes

- **To encapsulate internal state** and prevent external mutation.
- **To hide implementation details** from the public API.
- **To introduce nominal identity** between classes with private members.
- **To enforce invariants** by controlling access to sensitive fields.

### Syntax Rules and Structure

**General syntax:**

```
private memberName: Type;
```

**Component breakdown:**

- `private` — visibility modifier.
- `memberName` — identifier.

**Syntax rules.**

- `private` members are accessible only within the class body.
- `private` can combine with `readonly` and `static`.
- Private parameter properties create private fields.
- Private members affect structural compatibility nominally.
- Cross-instance access within the same class is allowed.

**Constraints and limitations.**

- `private` is **compile-time only**; it can be bypassed at runtime via bracket notation or `as any` [16†L43-L46].
- `private` does not prevent subclassing, but subclasses cannot access the private member.
- For true runtime privacy, use `#privateField` syntax.
- Google’s style guide recommends using TypeScript’s `private` modifier over `#` private fields, citing emit size and performance concerns when down-leveling [8†L29-L38]. Other guides, such as the Chromium style guide, take the opposite position and recommend `#` fields for runtime privacy [4†L43-L47]. This is a **context-dependent** choice.

### Annotated Complete Step by Step Code Examples

#### Example 1: Private field and method

```typescript
// example_private.ts
class BankAccount {
  private balance: number = 0;

  deposit(amount: number): void {
    if (amount <= 0) throw new Error("Invalid amount");
    this.balance += amount;
  }

  getBalance(): number {
    return this.balance;
  }
}

const account = new BankAccount();
account.deposit(100);
console.log(account.getBalance()); // Expected: 100

// account.balance; // Error: Property 'balance' is private.
```

**Expected Output:**
```
100
```

**Why this output occurs.** `balance` is private, so external code cannot read or write it directly. `deposit` controls mutation and validates the amount. `getBalance` provides controlled read access. The compiler enforces the private boundary.

---

#### Example 2: Private members and nominal compatibility

```typescript
// example_private_nominal.ts
class A {
  private secret = 1;
  value = 2;
}

class B {
  private secret = 1;
  value = 2;
}

const a = new A();
// const b: B = a;
// Error: Property 'secret' is private in type 'A' but not in type 'B'.
```

**Expected Output:**
```
(no output; compile-time error if uncommented)
```

**Why this output occurs.** Although `A` and `B` have identical public shapes, their `private` members originate from different declarations. TypeScript therefore considers them incompatible, exhibiting **nominal-like** behavior. Without the private members, they would be structurally compatible [5†L31-L38].

---

#### Example 3: Cross-instance private access

```typescript
// example_private_cross.ts
class Counter {
  private count = 0;

  increment(): void {
    this.count++;
  }

  // Cross-instance private access is allowed.
  isEqualTo(other: Counter): boolean {
    return this.count === other.count;
  }
}

const a = new Counter();
const b = new Counter();
a.increment();
console.log(a.isEqualTo(b)); // Expected: false
console.log(a.isEqualTo(a)); // Expected: true
```

**Expected Output:**
```
false
true
```

**Why this output occurs.** TypeScript allows a method of `Counter` to access the `private` `count` member of **another `Counter` instance**. This is a deliberate design choice; languages like Java and C# also permit it, while Ruby does not [16†L25-L31].

---

### Real-World Cases

- **Encapsulated domain state**: `private balance` with controlled mutation.
- **Internal helper methods**: `private validate()` hidden from the public API.
- **Nominal branding**: private members to distinguish semantically different types.

---

## 3. `protected`

### Definitions

**Core Definition.** **`protected`** allows access **within the declaring class and all subclasses**, but not from external code.

**Technical Definition.** The `protected` modifier is checked at compile time. It permits access from derived classes via `this` and `super`, but not from unrelated code. Like `private`, `protected` members introduce **nominal-like** compatibility: two classes with `protected` members are compatible only if the member comes from the **same declaration** [5†L35-L38]. TypeScript does **not** allow accessing `protected` members of a **sibling class** in a hierarchy (e.g., `Derived2` cannot access `protected x` through a `Derived1` instance) [17†L32-L40]. Derived classes may **increase** the visibility of a `protected` member to `public` unless the base class explicitly marks it as protected and the derived class repeats the modifier.

**Beginner-Friendly Explanation.** `protected` means **“this class and its children can use it, but outsiders cannot.”** It is useful when you want subclasses to customize behavior without exposing the internals to the world.

### Purposes

- **To share implementation details with subclasses** while hiding them from external code.
- **To enable template-method patterns** where subclasses override protected hooks.
- **To maintain encapsulation** across an inheritance hierarchy.

### Syntax Rules and Structure

**General syntax:**

```
protected memberName: Type;
```

**Component breakdown:**

- `protected` — visibility modifier.
- `memberName` — identifier.

**Syntax rules.**

- `protected` members are accessible within the class and subclasses.
- A subclass can access a protected member via `this` or `super`.
- Protected members affect structural compatibility nominally.
- `protected` can combine with `readonly` and `static`.
- Sibling-class access to protected members is **not** allowed.

**Constraints and limitations.**

- Protected members are **compile-time only**.
- External code cannot access protected members, even via subclass instances.
- Protected constructors prevent direct instantiation but allow subclassing.
- Derived classes can widen protected visibility to public; be careful to repeat the `protected` modifier if exposure is not intentional [17†L15-L31].

### Annotated Complete Step by Step Code Examples

#### Example 1: Protected member in an inheritance hierarchy

```typescript
// example_protected.ts
class Animal {
  protected energy = 100;

  protected consume(amount: number): void {
    this.energy -= amount;
  }
}

class Dog extends Animal {
  bark(): void {
    this.consume(10); // Allowed: protected access from subclass.
  }

  status(): string {
    return `Energy: ${this.energy}`;
  }
}

const dog = new Dog();
dog.bark();
console.log(dog.status()); // Expected: Energy: 90

// dog.energy;   // Error: Property 'energy' is protected.
// dog.consume(5); // Error: Property 'consume' is protected.
```

**Expected Output:**
```
Energy: 90
```

**Why this output occurs.** `energy` and `consume` are protected, so `Dog` (a subclass) can access them inside its methods. External code cannot access them directly. `bark` calls `consume(10)`, reducing energy to `90`.

---

#### Example 2: Sibling-class protected access is rejected

```typescript
// example_protected_sibling.ts
class Base {
  protected x: number = 1;
}

class Derived1 extends Base {
  protected x: number = 5;
}

class Derived2 extends Base {
  f1(other: Derived2) {
    other.x = 10; // OK: same class.
  }
  // f2(other: Derived1) {
  //   other.x = 10;
  // }
  // Error: Property 'x' is protected and only accessible within class 'Base' and its subclasses.
}

console.log("Sibling access is rejected");
```

**Expected Output:**
```
Sibling access is rejected
```

**Why this output occurs.** `Derived2` can access `protected x` through a `Derived2` instance, but **not** through a `Derived1` instance. TypeScript does not allow sibling-class protected access, following the same design as C# [17†L32-L40].

---

### Real-World Cases

- **Template method pattern**: protected hooks for subclasses to override.
- **Base service classes**: protected helper methods shared by subclasses.
- **Framework base controllers**: protected lifecycle methods.

---

## 4. `readonly`

### Definitions

**Core Definition.** **`readonly`** is a modifier that prevents **reassignment** of a property after its initial assignment. It can be applied to fields and parameter properties.

**Technical Definition.** A `readonly` property may be assigned only in its **declaration** or in the **constructor** of the declaring class [18†L1-L10]. TypeScript checks assignments at compile time and rejects reassignment. `readonly` does **not** perform a deep freeze; it prevents reassignment of the property binding, not mutation of the referenced object. `readonly` can be combined with `public`, `private`, `protected`, and `static`. Google’s style guide recommends: “Mark properties that are never reassigned outside of the constructor with the `readonly` modifier (these need not be deeply immutable)” [8†L39-L41]. The modifier is shallow; for read-only arrays, use `readonly T[]` or `ReadonlyArray<T>`.

**Beginner-Friendly Explanation.** `readonly` means **“set it once, then leave it alone.”** You can assign it when you declare it or in the constructor, but after that, TypeScript will not let you change it. It does not make the object inside immutable; it only prevents you from pointing the property at a different value.

### Purposes

- **To enforce immutability of identity** (IDs, creation timestamps).
- **To prevent accidental reassignment** of configuration values.
- **To document intent** that a property should not change after construction.
- **To enable safer refactoring** by making accidental mutations compile-time errors.

### Syntax Rules and Structure

**General syntax:**

```
readonly memberName: Type = value;
```

**Component breakdown:**

- `readonly` — modifier.
- `memberName` — identifier.
- `: Type` — annotation.
- `= value` — optional initializer; assignment here is allowed.

**Syntax rules.**

- `readonly` fields can be assigned at declaration or in the constructor.
- `readonly` parameter properties create readonly fields.
- `readonly` can combine with visibility modifiers and `static`.
- `readonly` does not prevent mutation of object contents (`readonly arr: number[]` still allows `arr.push(1)`).
- For read-only arrays, use `readonly T[]` or `ReadonlyArray<T>`.

**Constraints and limitations.**

- `readonly` is **compile-time only**; it does not enforce runtime immutability.
- `readonly` on arrays and objects is **shallow**; nested objects remain mutable.
- `readonly` cannot be applied to methods.
- Some style guides caution against overusing `readonly` on every property, suggesting it is most valuable for arrays and for matching external interfaces [1†L21-L25].

### Annotated Complete Step by Step Code Examples

#### Example 1: Readonly field assignment rules

```typescript
// example_readonly.ts
class Config {
  readonly host: string = "localhost";
  readonly port: number;

  constructor(port: number) {
    this.port = port; // Allowed: assignment in constructor.
  }

  // changeHost() {
  //   this.host = "example.com";
  // }
  // Error: Cannot assign to 'host' because it is a read-only property.
}

const config = new Config(3000);
console.log(config.host); // Expected: localhost
console.log(config.port); // Expected: 3000
```

**Expected Output:**
```
localhost
3000
```

**Why this output occurs.** `host` is initialized at declaration and cannot be reassigned afterward. `port` is readonly but uninitialized at declaration; the constructor assigns it once. The compiler rejects any assignment outside the constructor.

---

#### Example 2: Readonly parameter properties

```typescript
// example_readonly_param.ts
class Point {
  constructor(public readonly x: number, public readonly y: number) {}
}

const p = new Point(1, 2);
console.log(p.x); // Expected: 1

// p.x = 10; // Error: Cannot assign to 'x' because it is a read-only property.
```

**Expected Output:**
```
1
```

**Why this output occurs.** The parameter properties `public readonly x` and `public readonly y` create public readonly fields and assign the constructor arguments. After construction, the fields cannot be reassigned.

---

#### Example 3: Shallow readonly limitation

```typescript
// example_readonly_shallow.ts
class Container {
  readonly items: string[] = [];
}

const c = new Container();
c.items.push("a"); // Allowed: mutating the array, not reassigning the field.
console.log(c.items); // Expected: [ 'a' ]

// c.items = []; // Error: Cannot assign to 'items' because it is a read-only property.
```

**Expected Output:**
```
[ 'a' ]
```

**Why this output occurs.** `readonly` prevents reassignment of `items`, but does not prevent mutation of the array object it points to. For true read-only arrays, the type should be `readonly string[]`.

---

### Real-World Cases

- **Entity IDs**: `readonly id: string` assigned once.
- **Configuration values**: `readonly apiUrl: string`.
- **Immutable value objects**: `readonly x`, `readonly y` in `Point`.
- **React props and hook returns**: marking data as `readonly` documents intent and prevents accidental mutation [1†L5-L8].

---

## 5. Access Modifier Best Practices

### Definitions

**Core Definition.** **Access modifier best practices** are guidelines for choosing visibility and mutability modifiers that maximize **encapsulation, safety, clarity, and maintainability**, while avoiding redundant annotations, unnecessary exposure, and misuse of compile-time privacy as a security mechanism.

**Technical Definition.** Best practices center on the **Principle of Least Privilege**: use the **narrowest visibility** that satisfies the design. Google’s TypeScript Style Guide states: “TypeScript symbols are public by default. Never use the `public` modifier except when declaring non-readonly public parameter properties (in constructors)” and “Restrict visibility with `private` or `protected` where possible” [8†L5-L12][11†L16-L18]. `readonly` should be used for properties that are never reassigned outside the constructor [8†L39-L41]. The choice between TypeScript’s `private` and ECMAScript `#` private fields is **context-dependent**: TypeScript `private` is compile-time only and may offer better emit performance when down-leveling, while `#` provides **runtime-enforced** privacy [8†L29-L38][4†L43-L47]. Teams should make a deliberate, documented choice.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep classes safe and clean. They include: only use `private` or `protected` where possible (narrowest access); never write `public` because it is already the default; use `readonly` for things that should never change after construction; and decide deliberately whether you need compile-time privacy (TypeScript `private`) or runtime privacy (`#`).

### Purposes

- **To maximize encapsulation** by defaulting to the narrowest visibility.
- **To prevent accidental mutation** with `readonly`.
- **To avoid redundant annotations** that add noise without benefit.
- **To make privacy intentions explicit** and consistent across a codebase.
- **To choose the right privacy mechanism** (compile-time vs. runtime) for the context.

### Syntax Rules and Structure

Best practices are conventions that interact with modifier syntax. The core rules are:

1. **Default to `private` or `protected`.** Restrict visibility wherever possible; make members public only when they are part of the intended API [11†L16-L18].
2. **Never write `public` except for non-readonly parameter properties.** Public is the default; explicit `public` is noise. Google’s style guide treats explicit `public` on ordinary members as a style violation [8†L5-L12].
3. **Use `readonly` for immutable identity.** Mark fields that are never reassigned outside the constructor as `readonly` [8†L39-L41].
4. **Use `#` private fields deliberately.** Choose `#` when runtime privacy is required; choose TypeScript `private` when compile-time checking suffices and emit performance matters [4†L43-L47][8†L29-L38].
5. **Do not rely on `private` for security.** TypeScript’s `private` can be bypassed at runtime; use `#` for true encapsulation.
6. **Prefer parameter properties** for simple constructor-assigned fields [8†L41-L44].
7. **Be consistent.** Follow project or organizational style guides rather than mixing conventions arbitrarily.

**Component breakdown (visibility decision tree):**

```
Should this member be accessible outside the class?
  ├── No → Should subclasses access it?
  │        ├── Yes → protected
  │        └── No  → private (or #private for runtime privacy)
  └── Yes → Is it part of the stable public API?
           ├── Yes → public (omit modifier for non-parameter properties)
           └── No  → Reconsider the design; prefer private
```

**Constraints and limitations.**

- Narrow visibility can make testing harder; some teams use bracket notation or test-specific accessors.
- `#` private fields have compatibility and emit-size trade-offs when targeting older JavaScript versions [8†L34-L38].
- `readonly` is shallow; it does not guarantee deep immutability.
- Style guides differ on `public` and `#`; follow the project’s chosen convention.

### Annotated Complete Step by Step Code Examples

#### Example 1: Least privilege and avoiding redundant `public`

```typescript
// example_best_visibility.ts
class UserService {
  // ✅ private: implementation detail.
  private cache = new Map<string, string>();

  // ✅ public by default (no modifier needed).
  getUser(id: string): string | undefined {
    return this.cache.get(id);
  }

  // ❌ Redundant public (style violation under Google guide).
  // public setUser(id: string, name: string): void {
  //   this.cache.set(id, name);
  // }

  // ✅ Correct: omit public.
  setUser(id: string, name: string): void {
    this.cache.set(id, name);
  }
}

const svc = new UserService();
svc.setUser("u1", "Ada");
console.log(svc.getUser("u1")); // Expected: Ada

// svc.cache; // Error: Property 'cache' is private.
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `cache` is private and hidden. `getUser` and `setUser` are public by default, so no `public` modifier is needed. External code can call `setUser` and `getUser` but cannot access `cache`. This follows the Principle of Least Privilege.

---

#### Example 2: `readonly` for immutable identity

```typescript
// example_best_readonly.ts
class Order {
  constructor(
    public readonly id: string,
    public readonly createdAt: Date,
    public status: "pending" | "shipped" | "delivered" = "pending"
  ) {}
}

const order = new Order("o1", new Date());
order.status = "shipped"; // Allowed: status is mutable.
console.log(order.status); // Expected: shipped

// order.id = "o2"; // Error: Cannot assign to 'id' because it is a read-only property.
```

**Expected Output:**
```
shipped
```

**Why this output occurs.** `id` and `createdAt` are readonly and cannot be reassigned after construction. `status` is mutable and can transition between states. This correctly models domain invariants: identity and creation time are immutable, while workflow state changes.

---

#### Example 3: Compile-time `private` vs. runtime `#` privacy

```typescript
// example_best_privacy_choice.ts
class SecureConfig {
  #apiKey: string;          // Runtime privacy (ECMAScript).
  private region: string;   // Compile-time privacy (TypeScript).

  constructor(apiKey: string, region: string) {
    this.#apiKey = apiKey;
    this.region = region;
  }

  getMaskedKey(): string {
    return `${this.#apiKey.slice(0, 3)}***`;
  }
}

const cfg = new SecureConfig("secret123", "us-east-1");
console.log(cfg.getMaskedKey()); // Expected: sec***

// (cfg as any).region; // Allowed at runtime — TypeScript private is compile-time only.
// (cfg as any).#apiKey; // SyntaxError — # fields are truly private at runtime.
```

**Expected Output:**
```
sec***
```

**Why this output occurs.** `#apiKey` is an ECMAScript private field and is **truly private at runtime**. `region` is a TypeScript `private` field and is erased to a normal property, accessible via `as any` or bracket notation. The choice depends on whether runtime privacy is required.

---

### Real-World Cases

- **Service layers**: private caches and helpers; public methods as the API surface.
- **Domain entities**: `readonly id` and `readonly createdAt`; mutable status fields.
- **Security-sensitive data**: `#` private fields for API keys, tokens, or passwords.
- **Library design**: narrow visibility to prevent consumers from depending on internals.

---

## References

- TypeScript Handbook — Classes (Public, private, and protected modifiers) - https://www.typescriptlang.org/docs/handbook/2/classes.html
- TypeScript 1.3 Release Notes — Protected Modifier - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-3.html
- TypeScript Documentation — JSDoc Property Modifiers (`@public`, `@private`, `@protected`, `@readonly`) - https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html
- Microsoft Learn — Access Modifiers in TypeScript Classes - https://learn.microsoft.com/en-us/training/modules/typescript-declare-instantiate-classes/5-access-modifiers
- Google TypeScript Style Guide — Visibility and `readonly` - https://google.github.io/styleguide/tsguide.html
- TypeScript ESLint — `explicit-member-accessibility` Rule - https://typescript-eslint.io/rules/explicit-member-accessibility/
- MDN — Private Class Features (`#` fields) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields
- InfoQ — TypeScript 3.8 Adds Private Fields and Top-Level Await - https://www.infoq.com/news/2020/02/typescript-3-8-release/
- Convex — Readonly in TypeScript: Best Practices - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-readonly
- Chromium DevTools TypeScript Style Guide — Private Fields - https://chromium.googlesource.com/chromium/src/+/main/styleguide/typescript.md