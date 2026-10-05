# TypeScript Discriminated Unions: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A discriminated union (also called a tagged union or algebraic data type) is a TypeScript pattern where a union of object types shares a common literal property—the discriminant—whose value uniquely identifies each member. This allows TypeScript's control flow analysis to narrow the union to a specific member based on the discriminant's value.

**Technical Definition**
A discriminated union is a union type in which every constituent type has a property with the same name (the discriminant) whose type is a distinct literal type (string, number, or boolean literal) or a unique primitive. TypeScript's control flow analysis uses checks on the discriminant property to narrow the union to the matching constituent. This narrowing enables type-safe access to constituent-specific properties, exhaustive checking via the `never` type, and compile-time verification that all cases are handled. Discriminated unions are TypeScript's primary mechanism for modeling algebraic data types, state machines, and finite-state representations.

**Beginner-Friendly Explanation**
A discriminated union is a way to say "this value is one of several specific shapes, and each shape has a tag that tells you which one it is." For example, a shape might be `{ kind: "circle", radius: number }` or `{ kind: "square", side: number }`. The `kind` property is the tag (discriminant). When you check `shape.kind === "circle"`, TypeScript knows the shape is a circle, so you can safely access `radius`. This pattern is incredibly powerful for modeling states, API responses, and any data that can take multiple forms. It makes invalid states impossible to represent, catching bugs at compile time instead of runtime.

### Key Characteristics

- **Shared discriminant**: Every union member has the same property name with a unique literal value.
- **Automatic narrowing**: Checking the discriminant narrows the union to the matching member.
- **Exhaustiveness checking**: The `never` type ensures all cases are handled.
- **Impossible states**: Invalid combinations of properties cannot be represented.
- **Compile-time safety**: TypeScript catches missing cases when new members are added.
- **Destructuring support**: TypeScript 4.6+ narrows destructured discriminants.
- **Composable**: Discriminants can be nested, combined with boolean flags, or use multiple properties.

### Prerequisites

- Basic knowledge of union types
- Familiarity with type narrowing and type guards
- Understanding of interfaces and type aliases
- Familiarity with control flow constructs (`switch`, `if`/`else`)

### Related Programming Areas

- **Type Theory**: Discriminated unions are sum types (tagged unions)
- **State Machines**: Modeling states and transitions
- **Algebraic Data Types**: Result/Either types, Option/Maybe types
- **Redux and State Management**: Actions and reducers
- **API Design**: Modeling success/error responses

### Core Concepts / Features

1. Discriminant Properties (Literal Types, Enums, Unique Primitives)
2. Narrowing Destructured Discriminant Properties
3. State Modeling and Solving the "Impossible States" Problem
4. Finite-State Representations in Asynchronous Workflows (Idle, Loading, Success, Error)
5. Complex Discriminants (Shared Boolean Flags, Nesting, Multiple Properties)


## 1. Discriminant Properties (Literal Types, Enums, Unique Primitives)

### Definitions

**Core Definition**
A discriminant property is a property that is present in every member of a union type with the same name, but whose type is a distinct literal (or unique primitive) in each member. TypeScript uses the discriminant to narrow the union when its value is checked.

**Technical Definition**
The discriminant property must satisfy two conditions: (1) it must exist on every member of the union, and (2) its type must be a literal type (string, number, or boolean literal) or a unique primitive (such as an enum member) that is distinct across members. TypeScript's control flow analysis recognizes equality checks (`===`), `switch` statements, and `in` operator checks on the discriminant and narrows the union accordingly. String literal discriminants are the most common; numeric literals and enums also work. Boolean discriminants work with two-member unions (true/false), but for more complex cases, string or numeric literals are preferred.

**Beginner-Friendly Explanation**
A discriminant property is a "tag" that tells you which variant of a union you're dealing with. The tag has the same name in every variant (like `kind` or `type`), but a different value in each. For example, a `Shape` union might use `kind: "circle"` and `kind: "square"`. When you check `shape.kind`, TypeScript knows which shape it is. The tag must be a literal type—not just `string`, but `"circle"`. This makes narrowing work. You can use strings, numbers, or booleans as discriminants, but strings are the most common and readable.

### Purposes

- To enable automatic narrowing of union types based on a shared property.
- To model mutually exclusive variants of a data structure.
- To provide a clear, readable tag for each variant.
- To support exhaustive checking via the `never` type.
- To make invalid state combinations unrepresentable.

### Syntax Rules and Structure

**General Syntax: String Literal Discriminant**

```typescript
type UnionName =
  | { kind: "variant1"; property1: Type1 }
  | { kind: "variant2"; property2: Type2 }
  | { kind: "variant3"; property3: Type3 };
```

**Component Breakdown**
- `kind`: The discriminant property (shared name across all members).
- `"variant1"`, `"variant2"`, `"variant3"`: Distinct literal values.

**General Syntax: Numeric Literal Discriminant**

```typescript
type Response =
  | { status: 200; data: string }
  | { status: 404; error: string }
  | { status: 500; error: string };
```

**Component Breakdown**
- Numeric literals serve as discriminants.

**General Syntax: Enum Discriminant**

```typescript
enum ActionType {
  Add = "ADD",
  Remove = "REMOVE",
}

type Action =
  | { type: ActionType.Add; payload: string }
  | { type: ActionType.Remove; id: number };
```

**Component Breakdown**
- Enum members are literal types and work as discriminants.

**Syntax Rules**

- The discriminant property must be present on every union member.
- The discriminant type must be a literal type (string, number, boolean) or unique primitive.
- The discriminant values must be unique across members.
- Equality checks (`===`, `!==`), `switch` statements, and `in` checks narrow the union.
- String literal discriminants are the most common and recommended.
- Boolean discriminants work for two-member unions.
- The discriminant property name must be the same across all members.

**Constraints and Limitations**

- The discriminant must be a literal type, not a general primitive.
- Discriminant values must be unique for reliable narrowing.
- Multiple discriminants are not supported (only one property is used for narrowing).
- Nested discriminants (discriminants inside nested objects) are not automatically narrowed.
- Excess property checking with discriminated unions can be strict.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: String Literal Discriminant

```typescript
// Step 1: Define a discriminated union with a string literal discriminant.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number }
  | { kind: "rectangle"; width: number; height: number };

// Step 2: Calculate area using the discriminant for narrowing.
function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      // shape is narrowed to { kind: "circle"; radius: number }
      return Math.PI * shape.radius ** 2;
    case "square":
      // shape is narrowed to { kind: "square"; sideLength: number }
      return shape.sideLength ** 2;
    case "rectangle":
      // shape is narrowed to { kind: "rectangle"; width: number; height: number }
      return shape.width * shape.height;
  }
}

// Step 3: Test all variants.
console.log(calculateArea({ kind: "circle", radius: 5 }).toFixed(2));  // "78.54"
console.log(calculateArea({ kind: "square", sideLength: 4 }));         // 16
console.log(calculateArea({ kind: "rectangle", width: 3, height: 6 })); // 18

// Step 4: Invalid combinations are caught.
// const invalid: Shape = { kind: "circle", sideLength: 4 };
// ❌ Error: 'sideLength' does not exist in type '{ kind: "circle"; radius: number; }'.
```

**Expected Output:**
```
78.54
16
18
```

**Why This Output Occurs:** The `kind` property is the discriminant. The `switch` statement checks `shape.kind`, and TypeScript narrows `shape` to the matching member in each case. The invalid combination fails because `sideLength` is not part of the circle variant.

#### Example 2: Enum Discriminant

```typescript
// Step 1: Define an enum for action types.
enum ActionType {
  Add = "ADD",
  Remove = "REMOVE",
  Update = "UPDATE",
}

// Step 2: Define a discriminated union using the enum.
type Action =
  | { type: ActionType.Add; payload: string }
  | { type: ActionType.Remove; id: number }
  | { type: ActionType.Update; id: number; newValue: string };

// Step 3: Process actions with exhaustive checking.
function handleAction(action: Action): string {
  switch (action.type) {
    case ActionType.Add:
      return `Adding: ${action.payload}`;
    case ActionType.Remove:
      return `Removing ID: ${action.id}`;
    case ActionType.Update:
      return `Updating ${action.id} to ${action.newValue}`;
    default:
      const _exhaustive: never = action;
      throw new Error(`Unhandled action: ${JSON.stringify(_exhaustive)}`);
  }
}

console.log(handleAction({ type: ActionType.Add, payload: "item" }));
// "Adding: item"
console.log(handleAction({ type: ActionType.Remove, id: 42 }));
// "Removing ID: 42"
console.log(handleAction({ type: ActionType.Update, id: 1, newValue: "new" }));
// "Updating 1 to new"
```

**Expected Output:**
```
Adding: item
Removing ID: 42
Updating 1 to new
```

**Why This Output Occurs:** The enum members `ActionType.Add`, `ActionType.Remove`, and `ActionType.Update` are literal types that serve as discriminants. The `switch` statement narrows `action` to each variant. The `never` check in the `default` case ensures exhaustiveness.

### Real-World Cases

**Case 1: Redux Actions**
Redux actions use discriminated unions with `type` as the discriminant, enabling type-safe reducers.

**Case 2: API Responses**
API responses use `status` or `kind` as the discriminant to distinguish between success, error, and loading states.

**Case 3: Event Systems**
Event systems use `type` as the discriminant to distinguish between different event shapes.

---

## 2. Narrowing Destructured Discriminant Properties

### Definitions

**Core Definition**
Narrowing destructured discriminants is a TypeScript 4.6+ feature that allows the compiler to narrow the types of destructured variables based on checks on other destructured variables from the same discriminated union. This works when the destructured variables are `const` and not reassigned.

**Technical Definition**
TypeScript 4.6 introduced control flow analysis for destructured discriminated unions. When a discriminated union is destructured into individual `const` variables, and a check is performed on the discriminant variable, TypeScript narrows the other destructured variables that belong to the same union member. This works for both `const` declarations and destructured function parameters that are never assigned to. The feature eliminates the need to keep the original object intact for narrowing, making destructuring patterns type-safe.

**Beginner-Friendly Explanation**
Before TypeScript 4.6, if you destructured a discriminated union into separate variables, TypeScript lost track of the relationship between them. For example, if you destructured `{ kind, payload }` from an `Action`, checking `kind === "NumberContents"` did not narrow `payload`—TypeScript treated them as independent variables. TypeScript 4.6 fixed this: now, checking the discriminant variable narrows the other destructured variables. This means you can write clean destructuring code and still get type safety. The variables must be `const` (or function parameters that are never reassigned) for this to work.

### Purposes

- To enable type-safe destructuring of discriminated unions.
- To write clean code without keeping the original object intact for narrowing.
- To use the discriminant to narrow related destructured variables.
- To reduce verbosity in pattern matching code.
- To work with function parameters that are destructured.

### Syntax Rules and Structure

**General Syntax: Destructured Discriminant Narrowing**

```typescript
type Action =
  | { kind: "NumberContents"; payload: number }
  | { kind: "StringContents"; payload: string };

function processAction(action: Action) {
  const { kind, payload } = action;
  if (kind === "NumberContents") {
    // payload is number (TS 4.6+)
    let num = payload * 2;
  } else if (kind === "StringContents") {
    // payload is string (TS 4.6+)
    const str = payload.trim();
  }
}
```

**Component Breakdown**
- `const { kind, payload } = action`: Destructures the union.
- `if (kind === "NumberContents")`: Checks the discriminant.
- TypeScript 4.6+ narrows `payload` based on the `kind` check.

**General Syntax: Destructured Function Parameter**

```typescript
function processAction({ kind, payload }: Action) {
  if (kind === "NumberContents") {
    // payload is number (TS 4.6+)
  }
}
```

**Component Breakdown**
- The parameter is destructured directly.
- Narrowing works for destructured parameters that are never assigned to.

**Syntax Rules**

- The discriminated union must be destructured into `const` variables.
- Function parameters destructured directly are also supported (if never reassigned).
- The discriminant variable must be checked with equality (`===`, `!==`) or `switch`.
- The related variables must not be reassigned before the check.
- TypeScript 4.6+ is required for this feature.
- The feature works with `switch` statements and `if`/`else` chains.

**Constraints and Limitations**

- Destructuring with `let` may not narrow as expected (reassignment resets narrowing).
- The feature does not work with nested destructuring in all cases.
- Narrowing is based on the discriminant, not on checks of non-discriminant properties.
- The original object must be destructured from the same union type.
- TypeScript 4.6+ is required.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Destructured Discriminant Narrowing

```typescript
// Step 1: Define a discriminated union.
type Action =
  | { kind: "NumberContents"; payload: number }
  | { kind: "StringContents"; payload: string }
  | { kind: "BooleanContents"; payload: boolean };

// Step 2: Destructure and narrow (TypeScript 4.6+).
function processAction(action: Action): string {
  const { kind, payload } = action;

  if (kind === "NumberContents") {
    // payload is narrowed to number
    return `Number: ${payload * 2}`;
  }
  if (kind === "StringContents") {
    // payload is narrowed to string
    return `String: ${payload.toUpperCase()}`;
  }
  // payload is narrowed to boolean
  return `Boolean: ${payload}`;
}

// Step 3: Test all variants.
console.log(processAction({ kind: "NumberContents", payload: 21 }));
// "Number: 42"
console.log(processAction({ kind: "StringContents", payload: "hello" }));
// "String: HELLO"
console.log(processAction({ kind: "BooleanContents", payload: true }));
// "Boolean: true"

// Step 4: Destructured function parameters also work.
function handleAction({ kind, payload }: Action): string {
  switch (kind) {
    case "NumberContents":
      return `Number: ${payload * 2}`;  // payload is number
    case "StringContents":
      return `String: ${payload.trim()}`;  // payload is string
    case "BooleanContents":
      return `Boolean: ${payload}`;  // payload is boolean
  }
}

console.log(handleAction({ kind: "NumberContents", payload: 10 }));
// "Number: 20"
```

**Expected Output:**
```
Number: 42
String: HELLO
Boolean: true
Number: 20
```

**Why This Output Occurs:** The destructured `kind` and `payload` variables are connected by TypeScript's control flow analysis. Checking `kind` narrows `payload` to the corresponding type. The `const` destructuring and the `switch` statement on `kind` enable the narrowing.

#### Example 2: Limitations and Workarounds

```typescript
// Step 1: Destructuring with let may lose narrowing.
type Result =
  | { type: "success"; data: string }
  | { type: "error"; message: string };

function processResult(result: Result): void {
  // Using let may reset narrowing
  let { type, data, message } = result;

  if (type === "success") {
    // data is string here, but only if not reassigned
    console.log(data.toUpperCase());
  }
}

// Step 2: Reassignment resets narrowing.
function processWithReassignment(result: Result): void {
  let { type, data, message } = result;

  if (type === "success") {
    // data is string here
    let modified = data;  // Copy to a new variable
    modified = "new value";  // Reassignment does not affect data
    console.log(modified);  // "new value"
  }
}

// Step 3: Use const for reliable narrowing.
function processResultConst(result: Result): void {
  const { type, data, message } = result;

  if (type === "success") {
    // data is string (const — narrowing preserved)
    console.log(data.toUpperCase());
  } else {
    // message is string
    console.log(message.toLowerCase());
  }
}

processResultConst({ type: "success", data: "hello" });  // "HELLO"
processResultConst({ type: "error", message: "FAILED" });  // "failed"
```

**Expected Output:**
```
HELLO
failed
```

**Why This Output Occurs:** The `const` destructuring preserves narrowing. The `let` version may lose narrowing if reassignment occurs. Using `const` is the recommended approach for destructured discriminated unions.

### Real-World Cases

**Case 1: Redux Reducers with Destructured Actions**
Redux reducers destructure actions and use the `type` discriminant to narrow the payload type.

**Case 2: API Response Handlers**
API response handlers destructure `{ status, data, error }` and narrow based on `status`.

**Case 3: Form Validation**
Form validation handlers destructure field results and narrow based on the validation status.

---

## 3. State Modeling and Solving the "Impossible States" Problem

### Definitions

**Core Definition**
Discriminated unions solve the "impossible states" problem by making invalid state combinations unrepresentable in the type system. Instead of using multiple optional properties or boolean flags (which allow contradictory combinations), each legal state is modeled as a distinct variant with exactly the data it needs.

**Technical Definition**
The "impossible states" problem arises when a type allows combinations of properties that are logically contradictory—for example, `{ loading: true, data: User, error: string }` where all three flags are set simultaneously. Discriminated unions solve this by defining each state as a separate type with its own required properties. A state machine modeled as a discriminated union has one variant per legal state, and the discriminant ensures mutual exclusivity. This approach is the structural-typing answer to the "make impossible states impossible" principle, popularized by Richard Feldman.

**Beginner-Friendly Explanation**
The "impossible states" problem is when your type allows combinations that shouldn't be possible—like having both `loading` and `error` set to `true` at the same time. Discriminated unions fix this by making each state its own type. Instead of `{ loading: boolean, data?: User, error?: string }`, you write `{ status: "loading" } | { status: "success"; data: User } | { status: "error"; error: string }`. Now you can't have a "success" state without data, or a "loading" state with an error. The compiler enforces that only legal combinations exist. This is one of the most powerful benefits of discriminated unions—they make bugs impossible to represent.

### Purposes

- To make illegal state combinations unrepresentable.
- To provide compile-time guarantees that only valid states exist.
- To eliminate defensive runtime checks for impossible combinations.
- To model state machines and workflows explicitly.
- To improve code clarity by making states self-documenting.

### Syntax Rules and Structure

**General Syntax: State Machine as Discriminated Union**

```typescript
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };
```

**Component Breakdown**
- Each state is a distinct variant with the discriminant `status`.
- Each variant carries only the data relevant to that state.

**General Syntax: State Transitions**

```typescript
type Action =
  | { type: "FETCH" }
  | { type: "SUCCESS"; data: T }
  | { type: "ERROR"; error: Error }
  | { type: "CANCEL" };

function reducer(state: State, action: Action): State {
  switch (state.status) {
    case "idle":
      switch (action.type) {
        case "FETCH": return { status: "loading" };
      }
    case "loading":
      switch (action.type) {
        case "SUCCESS": return { status: "success", data: action.data };
        case "ERROR": return { status: "error", error: action.error };
        case "CANCEL": return { status: "idle" };
      }
    // ...
  }
}
```

**Component Breakdown**
- The reducer switches on both state and action discriminants.
- Invalid transitions are impossible because of the type system.

**Syntax Rules**

- Each legal state is a distinct union variant.
- The discriminant identifies the state.
- Each variant carries only its own data.
- Transitions are modeled as actions (another discriminated union).
- The reducer switches on both state and action discriminants.
- TypeScript verifies that all states and transitions are handled.

**Constraints and Limitations**

- The union must be maintained as states change.
- Adding a new state requires updating the reducer and all consumers.
- Some complex state machines may benefit from dedicated libraries (XState).
- The pattern is more verbose than optional properties but much safer.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Data Fetching State Machine

```typescript
// Step 1: Define the state machine as a discriminated union.
type FetchState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

// Step 2: Define the actions.
type FetchAction<T> =
  | { type: "FETCH" }
  | { type: "SUCCESS"; data: T }
  | { type: "ERROR"; error: Error }
  | { type: "CANCEL" };

// Step 3: Implement the reducer.
function fetchReducer<T>(
  state: FetchState<T>,
  action: FetchAction<T>
): FetchState<T> {
  switch (state.status) {
    case "idle":
      if (action.type === "FETCH") {
        return { status: "loading" };
      }
      return state;

    case "loading":
      switch (action.type) {
        case "SUCCESS":
          return { status: "success", data: action.data };
        case "ERROR":
          return { status: "error", error: action.error };
        case "CANCEL":
          return { status: "idle" };
        default:
          return state;
      }

    case "success":
      if (action.type === "FETCH") {
        return { status: "loading" };
      }
      return state;

    case "error":
      if (action.type === "FETCH") {
        return { status: "loading" };
      }
      return state;
  }
}

// Step 4: Test state transitions.
let state: FetchState<string> = { status: "idle" };
state = fetchReducer(state, { type: "FETCH" });
console.log(state);  // { status: 'loading' }

state = fetchReducer(state, { type: "SUCCESS", data: "Hello" });
console.log(state);  // { status: 'success', data: 'Hello' }

state = fetchReducer(state, { type: "ERROR", error: new Error("Failed") });
console.log(state);  // { status: 'error', error: Error: Failed }
```

**Expected Output:**
```
{ status: 'loading' }
{ status: 'success', data: 'Hello' }
{ status: 'error', error: Error: Failed }
```

**Why This Output Occurs:** The `FetchState<T>` union models four legal states. The reducer switches on `state.status` and handles transitions based on the action type. Invalid states (e.g., loading with data) are impossible because the type system doesn't allow them.

#### Example 2: Impossible States with Flags vs. Discriminated Union

```typescript
// ===== IMPOSSIBLE STATES WITH FLAGS =====
interface FlagBasedState {
  isLoading: boolean;
  data?: string;
  error?: string;
}

// This allows impossible combinations:
const badState1: FlagBasedState = { isLoading: true, data: "Hello", error: "Failed" };
const badState2: FlagBasedState = { isLoading: false, data: "Hello", error: "Failed" };
// Both compile, but they represent impossible states.

// ===== DISCRIMINATED UNION (IMPOSSIBLE STATES) =====
type ProperState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: string }
  | { status: "error"; error: string };

// Only legal states are representable:
const idle: ProperState = { status: "idle" };
const loading: ProperState = { status: "loading" };
const success: ProperState = { status: "success", data: "Hello" };
const error: ProperState = { status: "error", error: "Failed" };

// Invalid combinations are compile errors:
// const invalid: ProperState = { status: "loading", data: "Hello" };
// ❌ Error: 'data' does not exist in type '{ status: "loading"; }'.

console.log(success.data);  // "Hello"
console.log(error.error);    // "Failed"
```

**Expected Output:**
```
Hello
Failed
```

**Why This Output Occurs:** The flag-based state allows contradictory combinations because each flag is independent. The discriminated union makes each state mutually exclusive—you can't have `data` in a loading state because the loading variant doesn't have a `data` property. The compiler catches invalid combinations.

### Real-World Cases

**Case 1: React Data Fetching**
React components use discriminated unions to model fetch states, ensuring that data is only accessed when the state is "success".

**Case 2: Form State Management**
Form state uses discriminated unions to model pristine, validating, valid, invalid, and submitting states.

**Case 3: Authentication Flows**
Authentication state uses discriminated unions for unauthenticated, authenticating, authenticated, and error states.

**Case 4: Payment Processing**
Payment state uses discriminated unions for idle, processing, succeeded, and failed states.

---

## 4. Finite-State Representations in Asynchronous Workflows (Idle, Loading, Success, Error)

### Definitions

**Core Definition**
Finite-state representations model asynchronous workflows (data fetching, mutations, authentication) as a finite set of states using discriminated unions. The canonical example is the four-state model: idle, loading, success, and error. Each state carries exactly the data relevant to it.

**Technical Definition**
Asynchronous workflows in TypeScript benefit from finite-state modeling because they prevent the "boolean soup" problem—using multiple booleans (`isLoading`, `isError`, `isSuccess`) to track state, which allows contradictory combinations. A discriminated union with a `status` discriminant models each state explicitly: `{ status: "idle" }`, `{ status: "loading" }`, `{ status: "success"; data: T }`, `{ status: "error"; error: Error }`. The reducer pattern (or `useReducer` in React) transitions between states based on actions. Exhaustive checking ensures all states are handled, and narrowing ensures that `data` is only accessible in the success state.

**Beginner-Friendly Explanation**
When you're fetching data from an API, you have several states: you haven't started yet (idle), you're waiting for a response (loading), you got data (success), or something went wrong (error). Instead of using three booleans (`isLoading`, `isError`, `isSuccess`) that can contradict each other, you use a discriminated union with a `status` field. Each state is a distinct variant that carries only what it needs—the success state has `data`, the error state has `error`, and the loading state has neither. This makes your code safer: you can't access `data` unless you're in the success state, and you can't accidentally show a spinner and an error at the same time.

### Purposes

- To model asynchronous workflows as a finite set of legal states.
- To eliminate contradictory boolean combinations.
- To ensure data is only accessed in the appropriate state.
- To enable exhaustive handling of all possible states.
- To provide a clear, self-documenting representation of async state.

### Syntax Rules and Structure

**General Syntax: Async State Discriminated Union**

```typescript
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };
```

**Component Breakdown**
- `status`: The discriminant.
- Each variant carries its own data (`data` for success, `error` for error).

**General Syntax: Rendering Based on State**

```typescript
function renderState<T>(state: AsyncState<T>): string {
  switch (state.status) {
    case "idle": return "Ready to load";
    case "loading": return "Loading...";
    case "success": return `Data: ${JSON.stringify(state.data)}`;
    case "error": return `Error: ${state.error.message}`;
  }
}
```

**Component Breakdown**
- The `switch` on `status` narrows to each state.
- Each case handles the state's specific data.

**Syntax Rules**

- The discriminant is typically named `status` or `state`.
- Each state variant carries only its relevant data.
- The reducer or handler switches on the discriminant.
- Exhaustive checking ensures all states are handled.
- Data access is type-safe: `data` is only available in the success state.
- The state machine can be extended with additional states (e.g., `cancelled`).

**Constraints and Limitations**

- Adding a new state requires updating all consumers.
- The union must be maintained as requirements change.
- Some workflows may need more complex state machines (XState).
- The pattern is more verbose than booleans but much safer.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Data Fetching State Machine

```typescript
// Step 1: Define the async state union.
type FetchState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

// Step 2: Define actions.
type FetchAction<T> =
  | { type: "FETCH" }
  | { type: "SUCCESS"; data: T }
  | { type: "ERROR"; error: Error }
  | { type: "CANCEL" };

// Step 3: Implement the reducer.
function fetchReducer<T>(state: FetchState<T>, action: FetchAction<T>): FetchState<T> {
  switch (state.status) {
    case "idle":
      if (action.type === "FETCH") return { status: "loading" };
      return state;

    case "loading":
      switch (action.type) {
        case "SUCCESS": return { status: "success", data: action.data };
        case "ERROR": return { status: "error", error: action.error };
        case "CANCEL": return { status: "idle" };
        default: return state;
      }

    case "success":
      if (action.type === "FETCH") return { status: "loading" };
      return state;

    case "error":
      if (action.type === "FETCH") return { status: "loading" };
      return state;
  }
}

// Step 4: Test transitions.
let state: FetchState<string> = { status: "idle" };
console.log(state);  // { status: 'idle' }

state = fetchReducer(state, { type: "FETCH" });
console.log(state);  // { status: 'loading' }

state = fetchReducer(state, { type: "SUCCESS", data: "Hello" });
console.log(state);  // { status: 'success', data: 'Hello' }

state = fetchReducer(state, { type: "ERROR", error: new Error("Failed") });
console.log(state);  // { status: 'error', error: Error: Failed }
```

**Expected Output:**
```
{ status: 'idle' }
{ status: 'loading' }
{ status: 'success', data: 'Hello' }
{ status: 'error', error: Error: Failed }
```

**Why This Output Occurs:** The `FetchState<T>` union models the four legal states. The reducer transitions between states based on actions. Exhaustive checking ensures all states are handled.

#### Example 2: React Data Fetching Hook

```typescript
// Step 1: Define the async state union.
type FetchState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

// Step 2: Custom hook using the discriminated union.
function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = React.useState<FetchState<T>>({ status: "idle" });

  React.useEffect(() => {
    if (!url) return;

    const fetchData = async () => {
      setState({ status: "loading" });
      try {
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        setState({ status: "success", data });
      } catch (error) {
        setState({
          status: "error",
          error: error instanceof Error ? error : new Error("Unknown error"),
        });
      }
    };

    fetchData();
  }, [url]);

  return state;
}

// Step 3: Component using the hook.
function UserProfile({ userId }: { userId: string }) {
  const state = useFetch<User>(`/api/users/${userId}`);

  switch (state.status) {
    case "idle":
      return React.createElement("div", null, "Ready to load user");
    case "loading":
      return React.createElement("div", null, "Loading user...");
    case "success":
      return React.createElement("div", null, `User: ${state.data.name}`);
    case "error":
      return React.createElement("div", null, `Error: ${state.error.message}`);
  }
}
```

**Expected Output:** No runtime output (React component rendering). The state transitions from idle to loading to success or error based on the fetch result.

**Why This Output Occurs:** The `useFetch` hook manages the async state using the discriminated union. The component switches on `state.status` and renders accordingly. The `data` property is only accessible in the success state, and `error` only in the error state.

### Real-World Cases

**Case 1: React Data Fetching Libraries**
Libraries like React Query and SWR model fetch state using discriminated unions, providing type-safe access to data, error, and loading states.

**Case 2: Redux Toolkit Async Thunks**
Redux Toolkit's `createAsyncThunk` models async state with `pending`, `fulfilled`, and `rejected` statuses as a discriminated union.

**Case 3: Form Submission**
Form submission state uses discriminated unions for idle, submitting, success, and error states.

**Case 4: Authentication Flows**
Authentication state uses discriminated unions for unauthenticated, authenticating, authenticated, and error states.

---

## 5. Complex Discriminants (Shared Boolean Flags, Nesting, Multiple Properties)

### Definitions

**Core Definition**
Complex discriminants extend the basic discriminated union pattern to handle cases where the discriminant is not a simple string literal. This includes using shared boolean flags, nested discriminants (discriminants inside nested objects), and multiple properties that together identify the variant.

**Technical Definition**
TypeScript's discriminated union narrowing is primarily designed for a single literal discriminant. However, several advanced patterns exist: (1) **Boolean discriminants** work for two-member unions (`true`/`false`); (2) **Nested discriminants** (a discriminant inside a nested object) are not automatically narrowed by TypeScript; (3) **Multiple discriminants** (using two or more properties together) are not directly supported—TypeScript narrows based on one property at a time. Workarounds include flattening the discriminant to the top level, using a unique symbol or brand, or combining `in` checks with equality checks. The TypeScript issue #39050 tracks improvements for excess property checking in these scenarios.

**Beginner-Friendly Explanation**
Complex discriminants are advanced patterns for cases where a simple string tag isn't enough. For example, you might want to use a boolean (`true`/`false`) as the discriminant, or you might have a discriminant nested inside another object. TypeScript handles boolean discriminants for two-member unions, but nested discriminants are not automatically narrowed—you need to flatten them to the top level. Multiple discriminants (using two properties together) are also not directly supported; TypeScript narrows based on one property at a time. These are edge cases, but knowing them helps you avoid frustration when your narrowing doesn't work as expected.

### Purposes

- To handle two-state variants with boolean discriminants.
- To understand the limitations of nested discriminants and work around them.
- To combine multiple properties for complex variant identification.
- To use `in` checks and equality checks together for narrowing.
- To recognize when flattening the discriminant is necessary.

### Syntax Rules and Structure

**General Syntax: Boolean Discriminant**

```typescript
type Result =
  | { success: true; data: string }
  | { success: false; error: string };
```

**Component Breakdown**
- `success`: Boolean discriminant (works for two-member unions).

**General Syntax: Nested Discriminant (Not Automatically Narrowed)**

```typescript
type Nested =
  | { outer: { kind: "a"; value: number } }
  | { outer: { kind: "b"; value: string } };

// TypeScript does not narrow based on outer.kind alone.
// Workaround: flatten the discriminant.
```

**Component Breakdown**
- Nested discriminants require flattening for automatic narrowing.

**General Syntax: Multiple Discriminants (Not Directly Supported)**

```typescript
// TypeScript narrows based on one discriminant at a time.
type Complex =
  | { kind: "a"; subKind: "x"; value: number }
  | { kind: "a"; subKind: "y"; value: string }
  | { kind: "b"; value: boolean };
```

**Component Breakdown**
- Multiple discriminants require nested checks or flattening.

**Syntax Rules**

- Boolean discriminants work for two-member unions (`true`/`false`).
- Nested discriminants are not automatically narrowed—flatten to the top level.
- Multiple discriminants are not directly supported—narrow one at a time.
- `in` checks can combine with equality checks for complex narrowing.
- Excess property checking may not catch all invalid combinations in complex cases.
- TypeScript issue #39050 tracks improvements for excess property checking.

**Constraints and Limitations**

- Boolean discriminants are limited to two-member unions.
- Nested discriminants require flattening for narrowing.
- Multiple discriminants require sequential narrowing.
- Excess property checking may not catch invalid combinations.
- TypeScript does not support narrowing based on multiple properties simultaneously.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Boolean Discriminant

```typescript
// Step 1: Define a two-member union with a boolean discriminant.
type Result =
  | { success: true; data: string }
  | { success: false; error: string };

// Step 2: Narrow based on the boolean.
function handleResult(result: Result): string {
  if (result.success) {
    // result is narrowed to { success: true; data: string }
    return `Data: ${result.data}`;
  }
  // result is narrowed to { success: false; error: string }
  return `Error: ${result.error}`;
}

console.log(handleResult({ success: true, data: "Hello" }));  // "Data: Hello"
console.log(handleResult({ success: false, error: "Failed" })); // "Error: Failed"

// Step 3: Boolean discriminants with optional properties.
type ApiResult =
  | { ok: true; value: string }
  | { ok: false; error: string; code: number };

function processResult(result: ApiResult): string {
  if (result.ok) {
    return result.value;  // ✅ value is string
  }
  return `${result.error} (${result.code})`;  // ✅ error and code are available
}

console.log(processResult({ ok: true, value: "Success" }));  // "Success"
console.log(processResult({ ok: false, error: "Not found", code: 404 }));
// "Not found (404)"
```

**Expected Output:**
```
Data: Hello
Error: Failed
Success
Not found (404)
```

**Why This Output Occurs:** The boolean `success` property is the discriminant. `if (result.success)` narrows to the true variant, and the `else` branch narrows to the false variant. Each variant has its own properties.

#### Example 2: Nested Discriminant Workaround

```typescript
// Step 1: Nested discriminant (does not narrow automatically).
type Nested =
  | { outer: { kind: "a"; value: number } }
  | { outer: { kind: "b"; value: string } };

// Step 2: Attempt to narrow — does not work.
function processNested(item: Nested): void {
  // if (item.outer.kind === "a") {
  //   item.outer.value.toFixed(2);  // ❌ Error: value is number | string
  // }
}

// Step 3: Workaround — flatten the discriminant.
type Flattened =
  | { kind: "a"; value: number }
  | { kind: "b"; value: string };

function processFlattened(item: Flattened): string {
  if (item.kind === "a") {
    return `Number: ${item.value.toFixed(2)}`;
  }
  return `String: ${item.value.toUpperCase()}`;
}

console.log(processFlattened({ kind: "a", value: 3.14159 }));  // "Number: 3.14"
console.log(processFlattened({ kind: "b", value: "hello" }));  // "String: HELLO"

// Step 4: Alternative workaround — use a type predicate.
function isNestedKindA(item: Nested): item is { outer: { kind: "a"; value: number } } {
  return item.outer.kind === "a";
}

function processWithPredicate(item: Nested): string {
  if (isNestedKindA(item)) {
    return `Number: ${item.outer.value.toFixed(2)}`;  // ✅ value is number
  }
  return `String: ${(item as { outer: { kind: "b"; value: string } }).outer.value.toUpperCase()}`;
}

console.log(processWithPredicate({ outer: { kind: "a", value: 42 } }));
// "Number: 42.00"
```

**Expected Output:**
```
Number: 3.14
String: HELLO
Number: 42.00
```

**Why This Output Occurs:** Nested discriminants do not narrow automatically. Flattening the discriminant to the top level enables narrowing. Alternatively, a type predicate can encapsulate the nested check.

### Real-World Cases

**Case 1: Boolean Flags in API Responses**
Boolean `success` or `ok` flags are common in API responses and work as discriminants for two-state results.

**Case 2: Nested Configuration**
Configuration objects with nested discriminants require flattening for type-safe narrowing.

**Case 3: Multi-Level State Machines**
Complex state machines with sub-states require sequential narrowing or flattening of discriminants.

---

## References

- TypeScript Handbook: Discriminated Unions — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook: Exhaustiveness Checking — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- TypeScript 4.6 Release Notes: Control Flow Analysis for Destructured Discriminated Unions — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-6.html
- TypeScript Playground: Discriminate Types — https://www.typescriptlang.org/play/typescript/meta-types/discriminate-types.ts.html
- Steve Kinney: Discriminated Unions — https://stevekinney.com/courses/react-typescript/typescript-discriminated-unions
- Convex: TypeScript Discriminated Unions Explained — https://www.convex.dev/typescript/advanced/type-operators-manipulation/typescript-discriminated-union
- Total TypeScript: Discriminated Unions — https://www.totaltypescript.com/discriminated-unions
- State Machines (Educative Course) — https://github.com/0xMahabub/educative.io_course_resources/blob/main/Advanced%20TypeScript%20Masterclass%20-%20Learn%20Interactively/23_State_Machines.pdf
- TypeScript Issue #39050: Excess Property Checking with Discriminated Unions — https://github.com/microsoft/TypeScript/issues/39050
- MDN: switch Statement — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch
- MDN: in Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in
- TypeScript ESLint: switch-exhaustiveness-check — https://typescript-eslint.io/rules/switch-exhaustiveness-check/