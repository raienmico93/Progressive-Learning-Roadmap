# TypeScript Exhaustiveness Checking: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Exhaustiveness checking is a compile-time technique that ensures every possible variant of a union type (or enum) is handled in a pattern-matching construct such as a `switch` statement or an `if`/`else` chain. It uses the `never` type—TypeScript's bottom type—to verify that no unhandled cases remain.

**Technical Definition**
Exhaustiveness checking in TypeScript leverages the `never` type, which represents values that should never occur. Because `never` is a subtype of every type but no type (except `never`) is assignable to it, assigning a value to a `never`-typed variable produces a compile error if the value's type is not `never`. This property is exploited in the `default` case of a `switch` or the final `else` of an `if`/`else` chain: if all union members have been handled, the remaining value's type is `never`, and the assignment succeeds. If a case is missing, the value retains its type, and the assignment fails with a compile error. This technique makes adding new union members a compile-time error at every exhaustive consumer, preventing silent fallthrough.

**Beginner-Friendly Explanation**
Exhaustiveness checking is a way to make TypeScript verify that you've handled every possible case in a union. You write a `switch` statement with a `default` case that assigns the remaining value to a variable of type `never`. If you've handled all cases, TypeScript knows the value can't exist in the `default` case (its type is `never`), and the assignment is fine. If you forgot a case, the value still has a type, and TypeScript gives you an error. This is incredibly useful: when you add a new variant to a union, TypeScript tells you every place that needs updating. It's like a compiler-enforced checklist.

### Key Characteristics

- **Bottom type foundation**: `never` is the bottom type, assignable to every type but with no values of its own.
- **Compile-time only**: Exhaustiveness checking has no runtime cost; it is erased during compilation.
- **Refactoring safety**: Adding a new union member triggers compile errors at every unhandled consumer.
- **Runtime fallback**: A `default` case that throws provides a runtime safety net alongside compile-time checking.
- **Multiple patterns**: Works with `switch` statements, `if`/`else` chains, and object lookup records.
- **Production pattern**: The `UnreachableCaseError` class provides descriptive runtime errors for unreachable branches.

### Prerequisites

- Basic knowledge of union types and discriminated unions
- Familiarity with type narrowing and control flow analysis
- Understanding of the `never` type
- Familiarity with `switch` and `if`/`else` constructs

### Related Programming Areas

- **Type Theory**: `never` is the bottom type (⊥) in type theory
- **Discriminated Unions**: Exhaustiveness checking is most useful with discriminated unions
- **Control Flow Analysis**: The engine that determines the remaining type in `default` cases
- **Refactoring Safety**: Compile-time errors guide safe union extension
- **Error Handling**: Runtime errors for truly unreachable code

### Core Concepts / Features

1. The `never` Type as a Bottom Type for Compile-Time Safety
2. Exhaustive `switch` and `if`/`else` Chain Formatting
3. Preventing Unhandled Cases via the `never` Assignment Trick
4. Refactoring-Safe Union Handling
5. Custom Build-Time Error Alerting Using a Dedicated `UnreachableCaseError` Runtime Class


## 1. The `never` Type as a Bottom Type for Compile-Time Safety

### Definitions

**Core Definition**
The `never` type is TypeScript's bottom type—a type that represents values that can never occur. It is a subtype of every other type, but no type (except `never` itself) is assignable to it. This unique position makes `never` the foundation for exhaustiveness checking.

**Technical Definition**
In type theory, the bottom type (often denoted as ⊥) is the type that has no values. In TypeScript, `never` is that bottom type. Every TypeScript type is a subtype of `never`, meaning `never` is assignable to every type. However, the reverse is not true: only `never` is assignable to `never`. The `never` type arises naturally in several contexts: functions that never return (always throw or loop infinitely), impossible type intersections (e.g., `string & number`), and unreachable code branches identified by control flow analysis. When a function is annotated with `never` as its return type, TypeScript verifies that the function's endpoint is genuinely unreachable. This property is the foundation for exhaustiveness checking: if a value's type has been narrowed to `never`, then all possible cases have been handled.

**Beginner-Friendly Explanation**
`never` is a special type that means "this can never happen." It's the opposite of `any` in a sense: `any` can be anything, while `never` can be nothing. Every type can be assigned to `never`? No—it's the other way around: `never` can be assigned to every type, but nothing (except `never`) can be assigned to `never`. This makes `never` perfect for exhaustiveness checking. If you check all possible types of a union and narrow to `never`, TypeScript knows you've handled everything. If you forgot a case, the type isn't `never`, and TypeScript gives you an error.

### Purposes

- To represent values that can never occur in a well-typed program.
- To serve as the foundation for exhaustiveness checking.
- To model functions that never return (always throw or loop forever).
- To represent impossible type intersections.
- To enable type-level programming with conditional and mapped types.

### Syntax Rules and Structure

**General Syntax: `never` Return Type**

```typescript
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {
    // never exits
  }
}
```

**Component Breakdown**
- `): never`: The return type annotation indicating the function never returns.
- The function body must throw or loop forever.

**General Syntax: `never` in Exhaustiveness Checking**

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled value: ${JSON.stringify(value)}`);
}
```

**Component Breakdown**
- `value: never`: The parameter accepts only `never`.
- Calling `assertNever(value)` compiles only if `value` is `never`.

**Syntax Rules**

- `never` is assignable to every type.
- No type (except `never`) is assignable to `never`.
- Functions with `never` return type must not have reachable endpoints.
- `T | never` simplifies to `T`.
- `T & never` simplifies to `never`.
- `never` is the default type for unreachable code branches.
- TypeScript 2.0 introduced the `never` type.

**Constraints and Limitations**

- `never` is erased at runtime; it has no runtime representation.
- Functions that throw but are not annotated with `never` may have their return type inferred as `void` for backward compatibility with JavaScript class override patterns.
- `never` cannot be used as a variable type for values that will be assigned.
- Using `never` incorrectly (e.g., annotating a function that does return) causes compile errors.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `never` as a Return Type

```typescript
// Step 1: Define a function that always throws — return type is never.
function fail(message: string): never {
  throw new Error(message);
}

// Step 2: Define a function with an infinite loop — return type is never.
function infiniteLoop(): never {
  while (true) {
    // never exits
  }
}

// Step 3: Use the never-returning function in a code path.
function processValue(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  if (typeof value === "number") {
    return value.toFixed(2);
  }
  // This point is unreachable — value is never.
  return fail("Unexpected type");
}

console.log(processValue("hello"));  // "HELLO"
console.log(processValue(42));       // "42.00"

// Step 4: Verify that never is assignable to string.
// fail() returns never, which is assignable to string.
```

**Expected Output:**
```
HELLO
42.00
```

**Why This Output Occurs:** The `fail` function never returns, so its return type is `never`. Since `never` is assignable to every type, TypeScript allows `return fail(...)` in a function declared to return `string`. This pattern is useful for handling logically unreachable code paths.

#### Example 2: `never` in Intersections

```typescript
// Step 1: Conflicting property types produce never.
type Conflicting = { value: string } & { value: number };
// value: string & number = never

// Step 2: The type is uninhabitable.
// const invalid: Conflicting = { value: "hello" };
// ❌ Error: Type 'string' is not assignable to type 'never'.

// Step 3: Literal absorption prevents some conflicts.
type Absorbed = { value: string } & { value: "hello" };
// value: string & "hello" = "hello"

const absorbed: Absorbed = { value: "hello" };  // ✅ Allowed
console.log(absorbed.value);  // "hello"

// Step 4: never absorbs the entire intersection.
type Uninhabitable = { name: string } & never;
// Uninhabitable is never
```

**Expected Output:**
```
hello
```

**Why This Output Occurs:** The intersection `string & number` produces `never` because no value can be both a string and a number. The intersection `string & "hello"` produces `"hello"` because the literal is a subtype of `string`. The intersection with `never` absorbs everything, making the type uninhabitable.

### Real-World Cases

**Case 1: Error Handling**
Functions that always throw are annotated with `never`, enabling their use in exhaustive `switch` statements and unreachable code paths.

**Case 2: Exhaustiveness Checking**
The `assertNever` helper uses `never` to verify that all union members are handled in pattern-matching constructs.

**Case 3: Type-Level Programming**
`never` is used in conditional types to filter out types and in mapped types to remove properties.

**Case 4: Unreachable Code Detection**
TypeScript's control flow analysis assigns `never` to code branches that cannot be reached, catching logic errors at compile time.


## 2. Exhaustive `switch` and `if`/`else` Chain Formatting

### Definitions

**Core Definition**
Exhaustive `switch` and `if`/`else` chains are control flow constructs that handle every possible variant of a union type, with a final `default` case (or `else` branch) that asserts the remaining type is `never`. These patterns ensure that no case is silently ignored.

**Technical Definition**
An exhaustive `switch` statement handles every case of a discriminated union or enum, with a `default` case that calls `assertNever` or assigns the remaining value to a `never`-typed variable. The `default` case serves two purposes: it provides a runtime error for truly unreachable states, and it produces a compile-time error if a case is missing. Exhaustive `if`/`else` chains follow the same pattern, with the final `else` branch containing the `never` check. TypeScript's control flow analysis narrows the type in the `default`/`else` branch to `never` when all cases are handled, enabling the compile-time check.

**Beginner-Friendly Explanation**
An exhaustive `switch` statement handles every possible case of a union. You list a `case` for each variant, and then add a `default` case that throws an error or calls a special function. If you've handled all cases, TypeScript knows the `default` case is unreachable, so the code compiles. If you forgot a case, TypeScript gives you an error in the `default` case. The same pattern works for `if`/`else` chains: end with an `else` that does the `never` check. This ensures you never accidentally forget to handle a case, and it makes your code refactoring-safe.

### Purposes

- To ensure every union member is handled in a pattern-matching construct.
- To catch missing cases at compile time when adding new union members.
- To provide a runtime error for truly unreachable states.
- To document the intention that a union is fully handled.
- To enable refactoring safety by flagging all exhaustive consumers.

### Syntax Rules and Structure

**General Syntax: Exhaustive `switch` with `assertNever`**

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled value: ${JSON.stringify(value)}`);
}

function processValue(value: UnionType): ReturnType {
  switch (value.kind) {
    case "variant1":
      return handleVariant1(value);
    case "variant2":
      return handleVariant2(value);
    case "variant3":
      return handleVariant3(value);
    default:
      return assertNever(value);
  }
}
```

**Component Breakdown**
- `switch (value.kind)`: Switches on the discriminant.
- Each `case` handles a specific variant.
- `default: return assertNever(value)`: The exhaustiveness check.

**General Syntax: Exhaustive `if`/`else` Chain**

```typescript
function processValue(value: UnionType): ReturnType {
  if (value.kind === "variant1") {
    return handleVariant1(value);
  } else if (value.kind === "variant2") {
    return handleVariant2(value);
  } else if (value.kind === "variant3") {
    return handleVariant3(value);
  } else {
    const _exhaustive: never = value;
    throw new Error(`Unhandled value: ${JSON.stringify(_exhaustive)}`);
  }
}
```

**Component Breakdown**
- Each `if`/`else if` handles a variant.
- The final `else` contains the `never` check.

**General Syntax: Exhaustive Object Lookup**

```typescript
const MESSAGES: Record<UnionType["kind"], string> = {
  variant1: "Message 1",
  variant2: "Message 2",
  variant3: "Message 3",
  // Adding a new kind causes a compile error here
};
```

**Component Breakdown**
- `Record<UnionType["kind"], string>` requires all keys to be present.

**Syntax Rules**

- The `default` case (or final `else`) must contain the `never` check.
- `assertNever` is the recommended helper function.
- The `never` check can be an assignment (`const _exhaustive: never = value`) or a function call (`assertNever(value)`).
- The `switch` discriminant must be a literal type for narrowing to work.
- Exhaustive `if`/`else` chains work when each branch uses equality checks on the discriminant.
- Object lookups with `Record<Union["kind"], T>` enforce exhaustiveness at the declaration site.

**Constraints and Limitations**

- The `default` case is required for exhaustiveness checking (a `switch` without `default` does not trigger the check).
- `if`/`else` chains require explicit equality checks for each case.
- The `never` check must be reachable in the type system (TypeScript must be able to narrow to `never`).
- `assertNever` is a runtime function that throws; it must be called to provide runtime safety.
- Exhaustive checks do not work with non-literal discriminants (e.g., `boolean` discriminants work for two-state unions).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Exhaustive `switch` with `assertNever`

```typescript
// Step 1: Define a discriminated union.
type PaymentMethod =
  | { kind: "card"; cardNumber: string }
  | { kind: "bank"; accountNumber: string }
  | { kind: "crypto"; walletAddress: string };

// Step 2: Define the assertNever helper.
function assertNever(value: never): never {
  throw new Error(`Unhandled payment method: ${JSON.stringify(value)}`);
}

// Step 3: Process payments with an exhaustive switch.
function processPayment(method: PaymentMethod): string {
  switch (method.kind) {
    case "card":
      return `Processing card ending in ${method.cardNumber.slice(-4)}`;
    case "bank":
      return `Processing bank transfer from ${method.accountNumber}`;
    case "crypto":
      return `Processing crypto payment to ${method.walletAddress}`;
    default:
      return assertNever(method);  // Compile error if a case is missing
  }
}

// Step 4: Test all variants.
console.log(processPayment({ kind: "card", cardNumber: "4111111111111234" }));
// "Processing card ending in 1234"
console.log(processPayment({ kind: "bank", accountNumber: "123456789" }));
// "Processing bank transfer from 123456789"
console.log(processPayment({ kind: "crypto", walletAddress: "0xABC123" }));
// "Processing crypto payment to 0xABC123"

// Step 5: Adding a new variant triggers a compile error.
// type PaymentMethod = ... | { kind: "paypal"; email: string };
// ❌ Error in processPayment: Argument of type '{ kind: "paypal"; email: string; }'
// is not assignable to parameter of type 'never'.
```

**Expected Output:**
```
Processing card ending in 1234
Processing bank transfer from 123456789
Processing crypto payment to 0xABC123
```

**Why This Output Occurs:** The `switch` statement handles all three variants of `PaymentMethod`. In the `default` case, TypeScript narrows `method` to `never` because all cases are handled. The `assertNever` call compiles. If a fourth variant is added, the `default` case receives the new variant, and the `assertNever` call fails with a compile error.

#### Example 2: Exhaustive `if`/`else` Chain

```typescript
// Step 1: Define a discriminated union.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

// Step 2: Process shapes with an exhaustive if/else chain.
function calculateArea(shape: Shape): number {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  } else if (shape.kind === "square") {
    return shape.side ** 2;
  } else if (shape.kind === "rectangle") {
    return shape.width * shape.height;
  } else {
    // Exhaustiveness check — shape is never here
    const _exhaustive: never = shape;
    throw new Error(`Unhandled shape: ${JSON.stringify(_exhaustive)}`);
  }
}

// Step 3: Test all variants.
console.log(calculateArea({ kind: "circle", radius: 5 }).toFixed(2));  // "78.54"
console.log(calculateArea({ kind: "square", side: 4 }));               // 16
console.log(calculateArea({ kind: "rectangle", width: 3, height: 6 })); // 18

// Step 4: The _exhaustive variable is never at runtime (throw is unreachable).
// Adding a new variant causes a compile error on the assignment.
```

**Expected Output:**
```
78.54
16
18
```

**Why This Output Occurs:** The `if`/`else` chain handles all three shape variants. In the final `else`, TypeScript narrows `shape` to `never`. The assignment `const _exhaustive: never = shape` compiles. If a new variant is added without a corresponding `else if`, the assignment fails with a compile error.

### Real-World Cases

**Case 1: Redux Reducers**
Redux reducers use exhaustive `switch` statements on `action.type` to ensure all action types are handled, with `assertNever` catching missing cases.

**Case 2: State Machine Transitions**
State machines use exhaustive `switch` statements on state and event discriminants to ensure all transitions are handled.

**Case 3: API Response Handling**
API response handlers use exhaustive `switch` statements on `response.status` to handle all possible response states.

**Case 4: AST Visitors**
Compiler AST visitors use exhaustive `switch` statements on node types to ensure all node types are visited.


## 3. Preventing Unhandled Cases via the `never` Assignment Trick

### Definitions

**Core Definition**
The `never` assignment trick is the practice of assigning a value to a `never`-typed variable in the `default` case of a `switch` statement (or the final `else` of an `if`/`else` chain). If all union members are handled, the value's type is `never`, and the assignment succeeds. If a case is missing, the value retains its type, and TypeScript produces a compile error.

**Technical Definition**
The `never` assignment trick works by exploiting TypeScript's control flow analysis and the assignability rules of `never`. In the `default` case of an exhaustive `switch`, TypeScript narrows the discriminated union to `never` because all possible variants have been eliminated. Assigning this `never`-typed value to a `never`-typed variable (`const _exhaustive: never = value`) is valid. If a case is missing, the value's type includes the unhandled variant, which is not assignable to `never`, producing a compile error. This technique is the foundation for the `assertNever` helper and the `UnreachableCaseError` pattern.

**Beginner-Friendly Explanation**
The `never` assignment trick is a way to make TypeScript check that you've handled every case. You write `const _exhaustive: never = value` in the `default` case. If you've handled all cases, `value` is `never`, so the assignment works. If you forgot a case, `value` isn't `never` anymore, so TypeScript gives you an error. It's like a safety net that catches missing cases at compile time. The `assertNever` function does the same thing but throws an error at runtime, providing both compile-time and runtime safety.

### Purposes

- To enforce exhaustiveness checking without requiring a helper function.
- To provide a compile-time error when a case is missing.
- To serve as the foundation for the `assertNever` helper.
- To work with `if`/`else` chains as well as `switch` statements.
- To enable inline exhaustiveness checks in any pattern-matching construct.

### Syntax Rules and Structure

**General Syntax: Inline `never` Assignment**

```typescript
switch (value.kind) {
  case "variant1":
    return handleVariant1(value);
  case "variant2":
    return handleVariant2(value);
  default:
    const _exhaustive: never = value;
    throw new Error(`Unhandled value: ${JSON.stringify(_exhaustive)}`);
}
```

**Component Breakdown**
- `const _exhaustive: never = value`: The `never` assignment.
- If `value` is `never`, the assignment succeeds.
- If `value` is not `never`, the assignment fails with a compile error.

**General Syntax: `assertNever` Helper**

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled value: ${JSON.stringify(value)}`);
}

switch (value.kind) {
  case "variant1":
    return handleVariant1(value);
  case "variant2":
    return handleVariant2(value);
  default:
    return assertNever(value);
}
```

**Component Breakdown**
- `assertNever` accepts only `never`, providing the same check with a runtime throw.

**Syntax Rules**

- The `never` assignment must be in the `default` case (or final `else`).
- The variable should be named `_exhaustive` or `_exhaustiveCheck` by convention.
- The `throw` statement after the assignment provides runtime safety.
- The `assertNever` helper is preferred for reuse across multiple functions.
- The check works with discriminated unions, enums, and literal unions.
- The `never` assignment trick requires TypeScript 2.0 or later.

**Constraints and Limitations**

- The `never` assignment is a compile-time check; the variable is unused at runtime.
- The `throw` is unreachable when all cases are handled, but it's required for runtime safety.
- The `never` check must be in a reachable code path (TypeScript must be able to analyze it).
- The `assertNever` helper must be defined before use (or imported).
- The `never` assignment trick does not work with non-literal discriminants.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inline `never` Assignment

```typescript
// Step 1: Define a discriminated union.
type Status = "pending" | "processing" | "shipped" | "delivered";

// Step 2: Use the never assignment trick in a switch.
function getStatusMessage(status: Status): string {
  switch (status) {
    case "pending":
      return "Order received";
    case "processing":
      return "Preparing your order";
    case "shipped":
      return "On the way";
    case "delivered":
      return "Delivered";
    default:
      // Exhaustiveness check — status is never here
      const _exhaustive: never = status;
      throw new Error(`Unhandled status: ${_exhaustive}`);
  }
}

// Step 3: Test all cases.
console.log(getStatusMessage("pending"));    // "Order received"
console.log(getStatusMessage("processing")); // "Preparing your order"
console.log(getStatusMessage("shipped"));    // "On the way"
console.log(getStatusMessage("delivered"));  // "Delivered"

// Step 4: Adding a new status triggers a compile error.
// type Status = ... | "cancelled";
// ❌ Error: Type '"cancelled"' is not assignable to type 'never'.
```

**Expected Output:**
```
Order received
Preparing your order
On the way
Delivered
```

**Why This Output Occurs:** The `switch` handles all four status values. In the `default` case, TypeScript narrows `status` to `never`. The assignment `const _exhaustive: never = status` compiles. The `throw` is unreachable when all cases are handled, but it provides runtime safety if the union is extended without updating the switch.

#### Example 2: `assertNever` Helper with Multiple Functions

```typescript
// Step 1: Define the assertNever helper.
function assertNever(value: never, message?: string): never {
  throw new Error(message ?? `Unhandled value: ${JSON.stringify(value)}`);
}

// Step 2: Define a discriminated union.
type Action =
  | { type: "add"; payload: string }
  | { type: "remove"; id: number }
  | { type: "update"; id: number; newValue: string };

// Step 3: Use assertNever in multiple functions.
function handleAction(action: Action): string {
  switch (action.type) {
    case "add":
      return `Adding: ${action.payload}`;
    case "remove":
      return `Removing ID: ${action.id}`;
    case "update":
      return `Updating ${action.id} to ${action.newValue}`;
    default:
      return assertNever(action);
  }
}

function getActionPriority(action: Action): number {
  switch (action.type) {
    case "add":
      return 1;
    case "remove":
      return 2;
    case "update":
      return 3;
    default:
      return assertNever(action);
  }
}

// Step 4: Test both functions.
console.log(handleAction({ type: "add", payload: "item" }));  // "Adding: item"
console.log(getActionPriority({ type: "remove", id: 42 }));   // 2

// Step 5: Adding a new action type causes compile errors in both functions.
// type Action = ... | { type: "clear" };
// ❌ Error in handleAction: Argument of type '{ type: "clear"; }'
// is not assignable to parameter of type 'never'.
// ❌ Error in getActionPriority: same.
```

**Expected Output:**
```
Adding: item
2
```

**Why This Output Occurs:** The `assertNever` helper is reused across both functions. When a new action type is added, TypeScript produces compile errors in both `handleAction` and `getActionPriority`, ensuring both functions are updated. This is the primary benefit of the `assertNever` helper over inline `never` assignments.

### Real-World Cases

**Case 1: Shared Validation Helpers**
The `assertNever` helper is defined once and reused across multiple switch statements in a module, ensuring consistent exhaustiveness checking.

**Case 2: Redux Reducers with Multiple Action Handlers**
Redux reducers use `assertNever` to ensure all action types are handled, with the helper shared across the reducer and related functions.

**Case 3: Domain-Specific Exhaustiveness**
Domain-specific helpers (e.g., `assertNeverPaymentMethod`) can be created for specific union types, providing domain-specific error messages.

**Case 4: Test Assertions**
Test utilities use `assertNever` to verify that test cases cover all union variants, catching incomplete test suites.


## 4. Refactoring-Safe Union Handling

### Definitions

**Core Definition**
Refactoring-safe union handling is the property of exhaustive `switch` statements and `if`/`else` chains that causes compile errors when a new union member is added but not handled. This ensures that all consumers of a union are updated when the union is extended, preventing silent runtime failures.

**Technical Definition**
When a union type gains a new member, TypeScript's control flow analysis recognizes that the new member is not handled in existing `switch` statements (which use `assertNever` or the `never` assignment trick). The unhandled member's type is not assignable to `never`, producing a compile error at the `default` case. This compile error propagates to every exhaustive consumer of the union, forcing developers to update each one. Without exhaustiveness checking, adding a new union member would silently fall through to the `default` case, potentially causing runtime bugs. Exhaustiveness checking turns refactoring into a guided, compile-time-checked process.

**Beginner-Friendly Explanation**
Refactoring-safe union handling means that when you add a new variant to a union, TypeScript tells you every place that needs updating. Without exhaustiveness checking, adding a new variant might silently fall through to a `default` case, causing bugs. With exhaustiveness checking, TypeScript produces a compile error at every exhaustive `switch` or `if`/`else` chain, so you know exactly what to update. This makes refactoring unions safe and predictable. You can add a new variant with confidence, knowing the compiler will guide you to every consumer that needs updating.

### Purposes

- To catch missing cases at compile time when adding new union members.
- To guide developers to all consumers that need updating during refactoring.
- To prevent silent runtime failures from unhandled cases.
- To make union extension a compile-time-checked process.
- To reduce the risk of introducing bugs during refactoring.

### Syntax Rules and Structure

**General Syntax: Refactoring-Safe Switch**

```typescript
type Status = "pending" | "processing" | "shipped";

function getMessage(status: Status): string {
  switch (status) {
    case "pending": return "Pending";
    case "processing": return "Processing";
    case "shipped": return "Shipped";
    default: return assertNever(status);
  }
}

// Adding "delivered" to Status:
// type Status = "pending" | "processing" | "shipped" | "delivered";
// ❌ Error: Argument of type '"delivered"' is not assignable to 'never'.
```

**Component Breakdown**
- The `default` case with `assertNever` catches the new union member.
- The compile error appears at every exhaustive consumer.

**General Syntax: Refactoring-Safe Object Lookup**

```typescript
type Status = "pending" | "processing" | "shipped";

const STATUS_MESSAGES: Record<Status, string> = {
  pending: "Pending",
  processing: "Processing",
  shipped: "Shipped",
  // Adding "delivered" to Status causes a compile error here
};
```

**Component Breakdown**
- `Record<Status, string>` requires all keys to be present.
- Adding a new status causes a compile error at the record declaration.

**Syntax Rules**

- Every exhaustive consumer must use `assertNever` or the `never` assignment trick.
- The compile error appears at the `default` case when a new member is added.
- Object lookups with `Record<Union["kind"], T>` enforce exhaustiveness at the declaration site.
- Multiple consumers produce multiple compile errors, ensuring all are updated.
- The error message includes the unhandled member's type, making it easy to identify.

**Constraints and Limitations**

- Exhaustiveness checking only works if the `default` case is present.
- Non-exhaustive consumers (those without `assertNever`) will not produce compile errors.
- The compile error appears at the `default` case, not at the union declaration.
- Adding a new member to a union used in many places produces many compile errors (which is the intended behavior).
- Exhaustiveness checking does not catch missing cases in non-literal unions.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Refactoring-Safe Switch

```typescript
// Step 1: Define an initial union.
type OrderStatus = "pending" | "processing" | "shipped";

// Step 2: Define an exhaustive switch.
function getOrderMessage(status: OrderStatus): string {
  switch (status) {
    case "pending":
      return "Order received";
    case "processing":
      return "Preparing your order";
    case "shipped":
      return "On the way";
    default:
      return assertNever(status);
  }
}

// Step 3: Add a new union member.
type OrderStatusV2 = "pending" | "processing" | "shipped" | "delivered";

// Step 4: Update the function — compile error until the new case is added.
function getOrderMessageV2(status: OrderStatusV2): string {
  switch (status) {
    case "pending":
      return "Order received";
    case "processing":
      return "Preparing your order";
    case "shipped":
      return "On the way";
    // case "delivered":  // ← Uncomment to fix the error
    //   return "Delivered";
    default:
      return assertNever(status);
      // ❌ Error: Argument of type 'OrderStatusV2' is not assignable to 'never'.
      //   Type '"delivered"' is not assignable to type 'never'.
  }
}

// Step 5: The compile error tells exactly what's missing.
console.log("Add the 'delivered' case to fix the error.");
```

**Expected Output:** The code produces a compile error on the `assertNever(status)` call, indicating that `"delivered"` is not handled. This is the intended behavior.

**Why This Output Occurs:** When `"delivered"` is added to `OrderStatusV2`, TypeScript narrows `status` to `"delivered"` in the `default` case. Since `"delivered"` is not `never`, the `assertNever` call fails with a compile error. This forces the developer to handle the new variant.

#### Example 2: Multiple Consumers

```typescript
// Step 1: Define a union.
type ActionType = "add" | "remove" | "update";

// Step 2: Define multiple exhaustive consumers.
function handleAction(action: ActionType): string {
  switch (action) {
    case "add": return "Adding";
    case "remove": return "Removing";
    case "update": return "Updating";
    default: return assertNever(action);
  }
}

function getActionPriority(action: ActionType): number {
  switch (action) {
    case "add": return 1;
    case "remove": return 2;
    case "update": return 3;
    default: return assertNever(action);
  }
}

function isDestructive(action: ActionType): boolean {
  switch (action) {
    case "add": return false;
    case "remove": return true;
    case "update": return false;
    default: return assertNever(action);
  }
}

// Step 3: Add a new action type.
type ActionTypeV2 = "add" | "remove" | "update" | "clear";

// Step 4: All three consumers produce compile errors until updated.
// handleAction: ❌ Error on assertNever(action)
// getActionPriority: ❌ Error on assertNever(action)
// isDestructive: ❌ Error on assertNever(action)

// Step 5: The developer must update all three consumers.
console.log("Update all three functions to handle 'clear'.");
```

**Expected Output:** The code produces compile errors in all three functions, indicating that `"clear"` is not handled. This is the intended behavior.

**Why This Output Occurs:** Adding `"clear"` to the union causes all three exhaustive consumers to fail at their `default` cases. This ensures that no consumer silently ignores the new action type. The developer must update all three functions, preventing runtime bugs.

### Real-World Cases

**Case 1: Redux Action Types**
Adding a new action type to a Redux application triggers compile errors in all reducers, ensuring all reducers handle the new action.

**Case 2: API Response Variants**
Adding a new response variant (e.g., "rate_limited") triggers compile errors in all response handlers, ensuring all handlers are updated.

**Case 3: State Machine States**
Adding a new state to a state machine triggers compile errors in all transition handlers, ensuring all states are handled.

**Case 4: AST Node Types**
Adding a new AST node type triggers compile errors in all visitors, ensuring all visitors are updated to handle the new node type.

**Case 5: Feature Flags**
Adding a new feature flag value triggers compile errors in all feature flag consumers, ensuring all consumers are updated to handle the new flag.

---

## 5. Custom Build-Time Error Alerting Using a Dedicated `UnreachableCaseError` Runtime Class

### Definitions

**Core Definition**
The `UnreachableCaseError` pattern is a production-grade technique that uses a dedicated error class instead of a generic `assertNever` function. The class's constructor accepts only a `never`-typed value, providing the same compile-time exhaustiveness check while producing a descriptive runtime error that identifies the unhandled case.

**Technical Definition**
The `UnreachableCaseError` class extends `Error` and has a constructor parameter typed as `never`. In the `default` case of an exhaustive `switch`, calling `throw new UnreachableCaseError(value)` compiles only if `value` is `never` (i.e., all cases are handled). If a case is missing, the constructor's parameter type `never` rejects the argument, producing a compile error. At runtime, the error is thrown with a descriptive message that includes the unhandled value, making debugging easier. The `ts-essentials` library provides a production-ready `UnreachableCaseError` class, and the pattern is widely used in large TypeScript codebases.

**Beginner-Friendly Explanation**
The `UnreachableCaseError` pattern is like `assertNever` but more descriptive. Instead of calling a generic function, you throw a custom error class that's specifically designed for unreachable cases. The class's constructor only accepts `never`, so you get the same compile-time exhaustiveness check. At runtime, if the error is ever thrown (which shouldn't happen if all cases are handled), it includes the unhandled value in the error message, making it easy to debug. This is a production-grade pattern that's common in large codebases where descriptive errors are important.

### Purposes

- To provide the same compile-time exhaustiveness check as `assertNever`.
- To produce descriptive runtime errors that include the unhandled value.
- To use a dedicated error class for better error handling and logging.
- To follow a production-grade pattern used in large TypeScript codebases.
- To integrate with error monitoring tools that distinguish error classes.

### Syntax Rules and Structure

**General Syntax: `UnreachableCaseError` Class**

```typescript
class UnreachableCaseError extends Error {
  constructor(value: never) {
    super(`Unreachable case: ${JSON.stringify(value)}`);
    this.name = "UnreachableCaseError";
  }
}
```

**Component Breakdown**
- `extends Error`: Inherits from the built-in `Error` class.
- `constructor(value: never)`: The parameter accepts only `never`.
- `super(...)`: Calls the `Error` constructor with a descriptive message.

**General Syntax: Using `UnreachableCaseError`**

```typescript
switch (value.kind) {
  case "variant1":
    return handleVariant1(value);
  case "variant2":
    return handleVariant2(value);
  default:
    throw new UnreachableCaseError(value);
}
```

**Component Breakdown**
- `throw new UnreachableCaseError(value)`: Throws the custom error.
- Compiles only if `value` is `never`.

**Syntax Rules**

- The `UnreachableCaseError` class extends `Error`.
- The constructor parameter is typed as `never`.
- The error message includes the unhandled value.
- `throw new UnreachableCaseError(value)` replaces `assertNever(value)`.
- The class can be extended for domain-specific error types.
- The pattern works with `switch` statements and `if`/`else` chains.

**Constraints and Limitations**

- The `UnreachableCaseError` class must be defined before use.
- The constructor parameter must be `never` for the compile-time check to work.
- The error is thrown at runtime only if the unreachable case is reached.
- The class adds a small runtime overhead compared to `assertNever`.
- The pattern requires TypeScript 2.0+ for the `never` type.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `UnreachableCaseError`

```typescript
// Step 1: Define the UnreachableCaseError class.
class UnreachableCaseError extends Error {
  constructor(value: never) {
    super(`Unreachable case: ${JSON.stringify(value)}`);
    this.name = "UnreachableCaseError";
  }
}

// Step 2: Define a discriminated union.
type LogOptions =
  | { action: "openBlogPage"; data: { startTime: number } }
  | { action: "likeVideo"; data: { videoSrc: string } }
  | { action: "closeBlogPage"; data: { readingTime: number } };

// Step 3: Use UnreachableCaseError in a switch.
function log(options: LogOptions): void {
  switch (options.action) {
    case "openBlogPage":
      console.log(`Opened blog page within ${options.data.startTime}s`);
      return;
    case "likeVideo":
      console.log(`Liked video with src "${options.data.videoSrc}"`);
      return;
    case "closeBlogPage":
      console.log(`Closed blog page, reading time: ${options.data.readingTime}s`);
      return;
    default:
      throw new UnreachableCaseError(options);
      // ^? never
  }
}

// Step 4: Test all cases.
log({ action: "openBlogPage", data: { startTime: 5 } });
// "Opened blog page within 5s"
log({ action: "likeVideo", data: { videoSrc: "video.mp4" } });
// 'Liked video with src "video.mp4"'
log({ action: "closeBlogPage", data: { readingTime: 120 } });
// "Closed blog page, reading time: 120s"

// Step 5: Adding a new action triggers a compile error.
// type LogOptions = ... | { action: "unlikeVideo"; data: { videoSrc: string } };
// ❌ Error: Argument of type '{ action: "unlikeVideo"; data: { videoSrc: string; }; }'
// is not assignable to parameter of type 'never'.
```

**Expected Output:**
```
Opened blog page within 5s
Liked video with src "video.mp4"
Closed blog page, reading time: 120s
```

**Why This Output Occurs:** The `UnreachableCaseError` class provides the same compile-time exhaustiveness check as `assertNever` but with a more descriptive error class. In the `default` case, `options` is narrowed to `never` when all cases are handled. The `throw new UnreachableCaseError(options)` compiles. If a new action is added, the constructor's `never` parameter rejects the argument, producing a compile error.

#### Example 2: Production Error Monitoring Integration

```typescript
// Step 1: Define a custom error class with monitoring support.
class UnreachableCaseError extends Error {
  public readonly value: unknown;

  constructor(value: never) {
    super(`Unreachable case: ${JSON.stringify(value)}`);
    this.name = "UnreachableCaseError";
    this.value = value;

    // Integrate with error monitoring (e.g., Sentry)
    if (typeof window !== "undefined" && (window as any).Sentry) {
      (window as any).Sentry.captureException(this);
    }
  }
}

// Step 2: Define a state machine union.
type FetchState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

// Step 3: Use UnreachableCaseError in a reducer.
function fetchReducer<T>(
  state: FetchState<T>,
  action: { type: "FETCH" } | { type: "SUCCESS"; data: T } | { type: "ERROR"; error: Error }
): FetchState<T> {
  switch (state.status) {
    case "idle":
      if (action.type === "FETCH") return { status: "loading" };
      return state;
    case "loading":
      switch (action.type) {
        case "SUCCESS": return { status: "success", data: action.data };
        case "ERROR": return { status: "error", error: action.error };
        default: return state;
      }
    case "success":
      if (action.type === "FETCH") return { status: "loading" };
      return state;
    case "error":
      if (action.type === "FETCH") return { status: "loading" };
      return state;
    default:
      throw new UnreachableCaseError(state);
  }
}

// Step 4: Test the reducer.
let state: FetchState<string> = { status: "idle" };
state = fetchReducer(state, { type: "FETCH" });
console.log(state);  // { status: 'loading' }
state = fetchReducer(state, { type: "SUCCESS", data: "Hello" });
console.log(state);  // { status: 'success', data: 'Hello' }
```

**Expected Output:**
```
{ status: 'loading' }
{ status: 'success', data: 'Hello' }
```

**Why This Output Occurs:** The `UnreachableCaseError` class includes monitoring integration, so any unreachable case is automatically reported to Sentry (or another monitoring tool). The reducer handles all state transitions, and the `default` case provides a compile-time exhaustiveness check with runtime error reporting.

### Real-World Cases

**Case 1: Large Codebases with Error Monitoring**
Large TypeScript codebases use `UnreachableCaseError` to integrate exhaustiveness checking with error monitoring tools like Sentry or Rollbar, ensuring unreachable cases are reported.

**Case 2: Redux Reducers with Logging**
Redux reducers use `UnreachableCaseError` to log unhandled action types with full context, making debugging easier.

**Case 3: Domain-Specific Error Hierarchies**
Domain-specific error hierarchies extend `UnreachableCaseError` for specialized error handling (e.g., `UnreachablePaymentMethodError`).

**Case 4: Library Development**
Libraries use `UnreachableCaseError` to provide descriptive errors to consumers, helping them identify when they've forgotten to handle a case.

**Case 5: Testing Utilities**
Testing utilities use `UnreachableCaseError` to verify that test suites cover all union variants, catching incomplete tests at compile time.

---

## References

- TypeScript Handbook: Exhaustiveness Checking — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- TypeScript Handbook: The `never` Type — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#the-never-type
- TypeScript 2.0 Release Notes (`never` Type) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html
- TypeScript Playground: Unknown and Never — https://www.typescriptlang.org/play/typescript/primitives/unknown-and-never.ts.html
- TypeScript Playground: Discriminate Types — https://www.typescriptlang.org/play/typescript/meta-types/discriminate-types.ts.html
- `ts-essentials` `UnreachableCaseError` — https://github.com/ts-essentials/ts-essentials/tree/main/lib/functions/unreachable-case-error
- `ts-essentials` Documentation — https://ts-essentials.org/ts-essentials/functions/unreachablecaseerror
- `@axhxrx/assert-never` (JSR) — https://jsr.io/@axhxrx/assert-never
- Egghead: Use TypeScript’s `never` Type for Exhaustiveness Checking — https://egghead.io/lessons/typescript-use-typescript-s-never-type-for-exhaustiveness-checking
- FullStory: Discriminated Unions and Exhaustiveness Checking — https://www.fullstory.com/blog/discriminated-unions-and-exhaustiveness-checking-in-typescript/
- Convex: TypeScript Switch Statements — https://www.convex.dev/typescript/advanced/type-operators-manipulation/typescript-switch-statements
- TypeScript ESLint: switch-exhaustiveness-check — https://typescript-eslint.io/rules/switch-exhaustiveness-check/
- Khan Academy `UnreachableCaseError` — https://khan.github.io/flow-to-ts/classes/unreachablecaseerror.html
- Effective TypeScript: Item 58 — Write Modern JavaScript