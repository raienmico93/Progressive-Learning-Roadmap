# TypeScript Enums: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An enum (short for "enumeration") is a TypeScript language feature that defines a named set of constant values. Enums allow developers to create a collection of related values under a single type, making code more readable and self-documenting by replacing magic numbers or strings with descriptive names.

**Technical Definition**
In TypeScript, an enum is a special type that compiles to a JavaScript object (for non-const enums) with named members that map to numeric or string values. TypeScript supports three kinds of enums: numeric enums (with auto-incrementing or explicit values), string enums (with string literal values), and heterogeneous enums (mixing numeric and string values). Numeric enums generate reverse mappings (value → name) in addition to forward mappings (name → value), while string enums do not. The `const enum` variant is fully inlined at compile time and produces no runtime object, but is incompatible with certain module configurations and modern bundler assumptions. Enums are one of the few TypeScript features that are not a type-level extension of JavaScript—they generate runtime code.

**Beginner-Friendly Explanation**
An enum is a way to give names to a set of related values. Instead of writing `0`, `1`, `2` in your code and trying to remember what each number means, you can write `Status.Active`, `Status.Inactive`, `Status.Pending`. TypeScript creates both the names and the underlying values for you. Enums come in three flavors: numeric (where values are numbers and TypeScript auto-increments them), string (where values are strings), and heterogeneous (a mix). There's also a special `const enum` that gets completely removed from your compiled code—the values are just pasted in wherever you use them. While enums are popular, many modern TypeScript developers prefer using literal unions or `as const` objects because they produce less JavaScript and align better with the language's structural typing.

### Key Characteristics

- **Runtime code generation**: Unlike most TypeScript features, enums generate actual JavaScript code (except `const enum`).
- **Numeric auto-incrementing**: Numeric enum members automatically increment from the previous value, starting at 0.
- **Reverse mappings**: Numeric enums include a reverse mapping from value to name; string enums do not.
- **Three variants**: Numeric, string, and heterogeneous enums.
- **`const enum` inlining**: `const enum` members are inlined at use sites and produce no runtime object.
- **`isolatedModules` compatibility**: `const enum` is not compatible with `isolatedModules` (used by Babel, SWC, and some bundlers).
- **Type-level vs value-level**: Enums exist in both type space and value space.

### Prerequisites

- Basic knowledge of JavaScript objects and property access
- Familiarity with TypeScript primitive types
- Understanding of TypeScript's type system (unions, literal types)
- Familiarity with `tsconfig.json` compiler options (`preserveConstEnums`, `isolatedModules`)

### Related Programming Areas

- **Literal Types**: String and numeric literal types are the foundation of enum alternatives.
- **Union Types**: Literal unions are a common enum alternative.
- **Discriminated Unions**: Enums are often used as discriminants in tagged unions.
- **Module Systems**: `const enum` behavior depends on module system and bundler configuration.
- **Type-Level Programming**: `keyof typeof` with `as const` objects emulates enum behavior.

### Core Concepts / Features

1. Numeric Enums and Auto-Incrementing
2. String Enums
3. Heterogeneous Enums
4. Reverse Mappings (and Why String Enums Don't Get Them)
5. `const enum` (Inlining Behavior)
6. Enum Alternatives (Literal Unions, `as const` Objects)
7. Trade-offs and Modern Community Best Practices


## 1. Numeric Enums and Auto-Incrementing

### Definitions

**Core Definition**
A numeric enum is a TypeScript enum whose members have numeric values. By default, the first member is assigned the value `0`, and each subsequent member increments by 1. Developers can override the starting value or assign specific values to individual members, with auto-incrementing resuming from the last explicit value.

**Technical Definition**
Numeric enums compile to a JavaScript object with two properties per member: a forward mapping from name to value, and a reverse mapping from value to name. The compiler assigns values based on the following rules: if a member has an explicit initializer, that value is used; otherwise, the value is the previous member's value plus one; if no previous member exists, the value is `0`. Numeric enums are assignable from `number` in non-strict scenarios, but this is considered unsafe. TypeScript 5.0 made enums a union of their members (a "union enum"), improving type checking significantly over previous versions.

**Beginner-Friendly Explanation**
A numeric enum is a set of named numbers. When you write `enum Status { Active, Inactive, Pending }`, TypeScript gives `Active` the value `0`, `Inactive` the value `1`, and `Pending` the value `2`—automatically. You can start from a different number if you want: `enum Status { Active = 1, Inactive, Pending }` gives `1`, `2`, `3`. You can also assign specific values to individual members. Under the hood, TypeScript creates an object with both the names and the numbers, so you can look up either direction.

### Purposes

- To define a set of named numeric constants that are related to each other.
- To replace magic numbers in code with descriptive names for improved readability.
- To leverage auto-incrementing for sequential values like IDs, priorities, or states.
- To provide a stable numeric representation for serialization or wire protocols.
- To enable type-safe assignment of named values without exposing raw numbers.

### Syntax Rules and Structure

**General Syntax: Auto-Incrementing Numeric Enum**

```typescript
enum EnumName {
  Member1,        // 0
  Member2,        // 1
  Member3,        // 2
}
```

**Component Breakdown**
- `enum`: The keyword introducing the enum declaration.
- `EnumName`: The name of the enum (PascalCase by convention).
- `Member1`, `Member2`, `Member3`: The enum members, separated by commas.
- Values are auto-assigned starting at `0` and incrementing by `1`.

**General Syntax: Custom Initializer**

```typescript
enum EnumName {
  Member1 = 10,   // 10
  Member2,        // 11
  Member3 = 20,   // 20
  Member4,        // 21
}
```

**Component Breakdown**
- Explicit initializers override the auto-incrementing value.
- After an explicit value, subsequent members continue incrementing from that value.

**General Syntax: Accessing Enum Members**

```typescript
const value: EnumName = EnumName.Member1;  // Forward mapping
const name: string = EnumName[10];          // Reverse mapping (numeric only)
```

**Component Breakdown**
- `EnumName.Member1`: Forward access via property syntax.
- `EnumName[10]`: Reverse access via index syntax (numeric enums only).

**Syntax Rules**

- The first member defaults to `0` if no initializer is provided.
- Subsequent members auto-increment by `1` from the previous member's value.
- Explicit initializers can be any constant numeric expression.
- Enum members are separated by commas (trailing commas are allowed).
- Numeric enums support reverse mapping (value → name).
- Numeric enum members are accessible via both dot notation and index notation.
- TypeScript 5.0+ treats enums as union types of their members for narrowing.

**Constraints and Limitations**

- Numeric enums are assignable from `number` in non-union-enum scenarios, allowing unsafe assignment of arbitrary numbers.
- Values can be computed at compile time but must be constant expressions.
- Reverse mapping can be surprising: `EnumName[0]` returns the name, not a value.
- Numeric enums with duplicate values create ambiguous reverse mappings.
- Numeric enums are less type-safe than string enums or literal unions.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Numeric Enum with Auto-Incrementing

```typescript
// Step 1: Declare a numeric enum without explicit values.
enum Direction {
  Up,     // 0
  Down,   // 1
  Left,   // 2
  Right,  // 3
}

// Step 2: Access enum members by name.
const myDirection: Direction = Direction.Up;
console.log(`Value: ${myDirection}`);  // 0

// Step 3: Use reverse mapping to get the name from the value.
console.log(`Name: ${Direction[0]}`);   // "Up"
console.log(`Name: ${Direction[2]}`);   // "Left"

// Step 4: Use in a function.
function move(direction: Direction): void {
  console.log(`Moving in direction: ${Direction[direction]}`);
}

move(Direction.Up);     // "Moving in direction: Up"
move(Direction.Right);  // "Moving in direction: Right"
```

**Expected Output:**
```
Value: 0
Name: Up
Name: Left
Moving in direction: Up
Moving in direction: Right
```

**Why This Output Occurs:** The enum members are auto-assigned values `0` through `3`. Accessing `Direction.Up` returns the numeric value `0`. Reverse mapping `Direction[0]` returns the string `"Up"`. The `move` function uses reverse mapping to display the name of the direction.

#### Example 2: Custom Initializers and Auto-Incrementing

```typescript
// Step 1: Declare a numeric enum with custom starting value.
enum HttpStatus {
  OK = 200,
  Created,           // 201
  Accepted,          // 202
  BadRequest = 400,
  Unauthorized,      // 401
  Forbidden,         // 403 is skipped — auto-increment continues from 401
  NotFound,          // 402? No — 401 + 1 = 402... let's verify.
}

// Step 2: Check the actual values.
console.log(`OK: ${HttpStatus.OK}`);                  // 200
console.log(`Created: ${HttpStatus.Created}`);        // 201
console.log(`Accepted: ${HttpStatus.Accepted}`);      // 202
console.log(`BadRequest: ${HttpStatus.BadRequest}`);  // 400
console.log(`Unauthorized: ${HttpStatus.Unauthorized}`); // 401
console.log(`Forbidden: ${HttpStatus.Forbidden}`);    // 402
console.log(`NotFound: ${HttpStatus.NotFound}`);      // 403

// Step 3: Reverse mapping works for custom values.
console.log(`Status 200: ${HttpStatus[200]}`);  // "OK"
console.log(`Status 402: ${HttpStatus[402]}`);  // "Forbidden"
```

**Expected Output:**
```
OK: 200
Created: 201
Accepted: 202
BadRequest: 400
Unauthorized: 401
Forbidden: 402
NotFound: 403
Status 200: OK
Status 402: Forbidden
```

**Why This Output Occurs:** The explicit initializer `OK = 200` sets the starting value. `Created` and `Accepted` auto-increment to `201` and `202`. The explicit initializer `BadRequest = 400` resets the counter. `Unauthorized` becomes `401`, and auto-incrementing continues from there. Reverse mapping works for all values.

### Real-World Cases

**Case 1: HTTP Status Codes**
HTTP status codes are a natural fit for numeric enums because they have meaningful numeric values (200, 404, 500) that must be preserved for the wire protocol. A numeric enum allows developers to write `HttpStatus.NotFound` instead of `404`.

**Case 2: Priority Levels**
Task management systems use numeric enums for priority levels (`Low = 0`, `Medium = 1`, `High = 2`) where numeric comparison is meaningful for sorting.

**Case 3: State Machines**
State machines with sequential states benefit from auto-incrementing numeric enums, where the numeric value represents the order of states.

---

## 2. String Enums

### Definitions

**Core Definition**
A string enum is a TypeScript enum whose members have string values. Unlike numeric enums, string enums do not auto-increment, and each member must be explicitly initialized with a string literal. String enums do not generate reverse mappings, and each member is a distinct type that is not assignable from arbitrary strings.

**Technical Definition**
String enums, introduced in TypeScript 2.4, compile to a JavaScript object with only forward mappings (name → value). Each enum member must have a string literal initializer; otherwise, a compile error occurs. String enums are more type-safe than numeric enums because they are not assignable from arbitrary strings (unless the string is asserted). Each string enum member is a subtype of `string` but is not interchangeable with other strings. TypeScript 5.0+ treats string enums as union types of their members, improving narrowing and exhaustiveness checking.

**Beginner-Friendly Explanation**
A string enum is a set of named strings. When you write `enum Color { Red = "RED", Green = "GREEN", Blue = "BLUE" }`, each member has the exact string value you specify. Unlike numeric enums, you can't skip the values—every member needs an explicit string. Also unlike numeric enums, you can't look up the name from the value (no reverse mapping). String enums are safer than numeric enums because TypeScript won't let you assign just any string to a string enum variable—only the specific enum members are allowed.

### Purposes

- To define a set of named string constants that are related to each other.
- To provide meaningful string values for serialization, logging, and API communication.
- To prevent accidental assignment of arbitrary strings to enum-typed variables.
- To improve debugging by using readable string values in logs and error messages.
- To enable type-safe string-based discriminated unions.

### Syntax Rules and Structure

**General Syntax: String Enum**

```typescript
enum EnumName {
  Member1 = "value1",
  Member2 = "value2",
  Member3 = "value3",
}
```

**Component Breakdown**
- `Member1 = "value1"`: Each member must have a string literal initializer.
- The values are the actual strings used at runtime.

**General Syntax: Accessing String Enum Members**

```typescript
const value: EnumName = EnumName.Member1;  // Forward mapping only
// No reverse mapping available.
```

**Component Breakdown**
- `EnumName.Member1`: Returns the string value `"value1"`.
- Reverse mapping is not available for string enums.

**Syntax Rules**

- Every member must have an explicit string literal initializer.
- String enums do not auto-increment (they can't—strings have no natural successor).
- String enums do not generate reverse mappings.
- String enum members are not assignable from arbitrary strings.
- String enums are assignable to `string`, but `string` is not assignable to the enum.
- TypeScript 5.0+ treats string enums as union types of their literal values.

**Constraints and Limitations**

- All members must have initializers; there is no auto-incrementing.
- Reverse mapping is not generated, so value-to-name lookup requires a manual map.
- String enums are not assignable from their literal values without an assertion (e.g., `"RED"` is not assignable to `Color.Red` without `as`).
- String enums increase bundle size compared to `const enum` or literal unions.
- Duplicate string values in the same enum are allowed but create aliases.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic String Enum

```typescript
// Step 1: Declare a string enum.
enum LogLevel {
  Debug = "DEBUG",
  Info = "INFO",
  Warn = "WARN",
  Error = "ERROR",
}

// Step 2: Use enum members.
const level: LogLevel = LogLevel.Info;
console.log(`Level: ${level}`);  // "INFO"

// Step 3: String enums are not assignable from arbitrary strings.
// const badLevel: LogLevel = "DEBUG";  // ❌ Error: Type '"DEBUG"' is not assignable to type 'LogLevel'.
const goodLevel: LogLevel = LogLevel.Debug;  // ✅

// Step 4: Logging with the enum.
function log(level: LogLevel, message: string): void {
  console.log(`[${level}] ${message}`);
}

log(LogLevel.Warn, "Low disk space");   // "[WARN] Low disk space"
log(LogLevel.Error, "Connection lost"); // "[ERROR] Connection lost"

// Step 5: No reverse mapping exists.
// console.log(LogLevel["DEBUG"]);  // ❌ Error: Property 'DEBUG' does not exist... wait, it does.
// Actually, LogLevel["DEBUG"] returns undefined because there's no reverse mapping.
console.log(LogLevel["DEBUG"]);  // undefined
```

**Expected Output:**
```
Level: INFO
[WARN] Low disk space
[ERROR] Connection lost
undefined
```

**Why This Output Occurs:** Each string enum member has an explicit string value. Assigning the literal string `"DEBUG"` to a `LogLevel` variable fails because string enums are nominal. Accessing `LogLevel["DEBUG"]` returns `undefined` because no property with that name exists—the property names are `Debug`, `Info`, etc., and the values are the strings. There is no reverse mapping.

#### Example 2: String Enum in a Discriminated Union

```typescript
// Step 1: Define a string enum for action types.
enum ActionType {
  Add = "ADD",
  Remove = "REMOVE",
  Update = "UPDATE",
}

// Step 2: Define action interfaces using the enum as a discriminant.
interface AddAction {
  type: ActionType.Add;
  payload: string;
}

interface RemoveAction {
  type: ActionType.Remove;
  id: number;
}

interface UpdateAction {
  type: ActionType.Update;
  id: number;
  newValue: string;
}

type Action = AddAction | RemoveAction | UpdateAction;

// Step 3: Use narrowing with the discriminant.
function handleAction(action: Action): void {
  switch (action.type) {
    case ActionType.Add:
      console.log(`Adding: ${action.payload}`);
      break;
    case ActionType.Remove:
      console.log(`Removing ID: ${action.id}`);
      break;
    case ActionType.Update:
      console.log(`Updating ID ${action.id} to ${action.newValue}`);
      break;
  }
}

handleAction({ type: ActionType.Add, payload: "New item" });
handleAction({ type: ActionType.Remove, id: 42 });
handleAction({ type: ActionType.Update, id: 7, newValue: "Updated" });
```

**Expected Output:**
```
Adding: New item
Removing ID: 42
Updating ID 7 to Updated
```

**Why This Output Occurs:** The string enum `ActionType` provides distinct literal types for each action type. The `type` property in each interface uses the specific enum member as a literal type, enabling discriminated union narrowing in the `switch` statement.

### Real-World Cases

**Case 1: API Error Codes**
APIs that return string error codes (e.g., `"INVALID_TOKEN"`, `"RATE_LIMITED"`) benefit from string enums, which preserve the exact strings while providing type safety.

**Case 2: Redux Action Types**
Redux action types are traditionally strings. String enums provide a type-safe alternative to plain string constants, reducing typos and enabling exhaustive checking.

**Case 3: Configuration Values**
Configuration values like environment names (`"development"`, `"staging"`, `"production"`) are naturally string enums, preserving the exact strings while preventing typos.

---

## 3. Heterogeneous Enums

### Definitions

**Core Definition**
A heterogeneous enum is a TypeScript enum that mixes numeric and string members in the same declaration. While technically allowed, heterogeneous enums are generally discouraged because they provide few benefits and introduce confusion.

**Technical Definition**
Heterogeneous enums allow both numeric and string initializers within the same enum declaration. The compiler permits this but does not generate reverse mappings for the string members (only numeric members get reverse mappings). TypeScript's official documentation explicitly states: "Unless you're really trying to take advantage of JavaScript's runtime behavior... it's advised that you don't do this." Heterogeneous enums are legal but are considered an anti-pattern in modern TypeScript.

**Beginner-Friendly Explanation**
A heterogeneous enum is an enum that has both numbers and strings in it. For example, `enum Mixed { A = 1, B = "B" }`. TypeScript allows this, but it's usually a bad idea. The numeric members get reverse mappings, but the string members don't, which creates inconsistent behavior. Most style guides recommend avoiding heterogeneous enums entirely—if you need mixed types, a different construct (like a union of literal types) is usually clearer.

### Purposes

- To technically allow mixing numeric and string values in a single enum when required by legacy code.
- To provide a transition mechanism when migrating from numeric to string enums.
- To demonstrate the edge case for educational purposes.
- To technically support JavaScript's runtime behavior where mixed properties are possible.
- To document the TypeScript feature for completeness even though it is discouraged.

### Syntax Rules and Structure

**General Syntax: Heterogeneous Enum**

```typescript
enum EnumName {
  NumericMember = 1,
  StringMember = "string",
  AnotherNumeric = 2,
  AnotherString = "another",
}
```

**Component Breakdown**
- Members can have either numeric or string initializers.
- Numeric members auto-increment from the previous numeric member.
- String members must have explicit string initializers.

**Syntax Rules**

- Heterogeneous enums allow both numeric and string members.
- Numeric members get reverse mappings; string members do not.
- Auto-incrementing applies only to numeric members.
- String members must have explicit initializers.
- TypeScript does not produce a compile error for heterogeneous enums.

**Constraints and Limitations**

- Heterogeneous enums have inconsistent reverse mapping behavior.
- Type safety is reduced because the enum's members have different types.
- Most style guides and linters discourage heterogeneous enums.
- TypeScript's official documentation advises against them.
- The `isolatedModules` and `const enum` interactions become more complex.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Heterogeneous Enum Behavior

```typescript
// Step 1: Declare a heterogeneous enum (discouraged, but legal).
enum Mixed {
  No = 0,
  Yes = "YES",
}

// Step 2: Access members.
console.log(`No: ${Mixed.No}`);     // 0
console.log(`Yes: ${Mixed.Yes}`);   // "YES"

// Step 3: Reverse mapping only works for numeric members.
console.log(`Reverse 0: ${Mixed[0]}`);  // "No"
// String members do not have reverse mappings:
console.log(`Reverse "YES": ${Mixed["YES"]}`);  // undefined

// Step 4: The type of Mixed.No is 0 (numeric literal).
// The type of Mixed.Yes is "YES" (string literal).
const numValue: Mixed.No = Mixed.No;    // ✅
const strValue: Mixed.Yes = Mixed.Yes;  // ✅

// Step 5: Assigning the wrong member type fails.
// const bad: Mixed.No = Mixed.Yes;  // ❌ Error
```

**Expected Output:**
```
No: 0
Yes: YES
Reverse 0: No
Reverse "YES": undefined
```

**Why This Output Occurs:** The heterogeneous enum contains both a numeric member (`No = 0`) and a string member (`Yes = "YES"`). Reverse mapping works for `0` (returns `"No"`) but not for `"YES"` (returns `undefined`). The types of the members are distinct, so `Mixed.No` and `Mixed.Yes` are not interchangeable.

### Real-World Cases

**Case 1: Legacy Migration**
When migrating from a numeric enum to a string enum, a heterogeneous enum can serve as an intermediate step. However, this is a temporary measure and should not be a permanent design.

**Case 2: JavaScript Interop**
When interfacing with JavaScript libraries that use mixed numeric and string constants, a heterogeneous enum may be the only way to type the values accurately.

**Case 3: Educational Examples**
Heterogeneous enums are primarily useful for demonstrating TypeScript's edge cases and understanding the language's boundaries.

---

## 4. Reverse Mappings (and Why String Enums Don't Get Them)

### Definitions

**Core Definition**
Reverse mapping is a feature of numeric enums where the compiled JavaScript object includes both forward mappings (name → value) and reverse mappings (value → name). String enums do not generate reverse mappings because multiple names could theoretically map to the same string value.

**Technical Definition**
When a numeric enum is compiled, TypeScript generates JavaScript code that assigns both `EnumName[EnumName.Member] = "Member"` and `EnumName.Member = value`. This creates a bidirectional mapping. For string enums, only the forward mapping is generated because reverse mapping from a string value to a name is ambiguous—multiple members could have the same string value, and there's no guarantee of uniqueness. Reverse mappings are also not generated for computed members (members with non-constant initializers).

**Beginner-Friendly Explanation**
Reverse mapping means you can look up an enum's name from its value. If `Status.Active` is `0`, then `Status[0]` gives you `"Active"`. This works for numeric enums because numbers are unique and auto-incrementing. But string enums don't get reverse mappings because if two members had the same string value, TypeScript wouldn't know which name to return. So with string enums, you can go from name to value, but not from value back to name.

### Purposes

- To enable value-to-name lookups in numeric enums for debugging and logging.
- To provide bidirectional access to enum members without a separate lookup table.
- To support code that receives numeric values (e.g., from APIs) and needs to display their names.
- To illustrate why string enums have different runtime behavior.
- To document the runtime shape of compiled enums for interoperability.

### Syntax Rules and Structure

**General Syntax: Reverse Mapping (Numeric Enums)**

```typescript
enum Status { Active, Inactive }
// Compiled JavaScript:
// var Status;
// (function (Status) {
//   Status[Status["Active"] = 0] = "Active";
//   Status[Status["Inactive"] = 1] = "Inactive";
// })(Status || (Status = {}));

const name: string = Status[0];  // "Active"
const value: number = Status.Active;  // 0
```

**Component Breakdown**
- `Status[Status["Active"] = 0] = "Active"`: Assigns `Active = 0` and `0 = "Active"` in one expression.
- Forward mapping: `Status.Active === 0`.
- Reverse mapping: `Status[0] === "Active"`.

**Syntax Rules**

- Numeric enums generate reverse mappings for all non-computed members.
- String enums do not generate reverse mappings.
- Heterogeneous enums generate reverse mappings only for numeric members.
- Computed members (with non-constant initializers) do not generate reverse mappings.
- Reverse mappings are `string` values (the member names).

**Constraints and Limitations**

- Reverse mappings increase the runtime size of numeric enums.
- Duplicate numeric values create ambiguous reverse mappings (the last member wins).
- Reverse mappings are not available for string enums.
- Reverse mappings are not available for computed enum members.
- The `const enum` variant does not generate reverse mappings (since it produces no runtime object).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reverse Mapping in Numeric Enums

```typescript
// Step 1: Declare a numeric enum.
enum Weekday {
  Monday,     // 0
  Tuesday,    // 1
  Wednesday,  // 2
  Thursday,   // 3
  Friday,     // 4
}

// Step 2: Forward mapping — name to value.
console.log(Weekday.Monday);     // 0
console.log(Weekday.Wednesday);  // 2

// Step 3: Reverse mapping — value to name.
console.log(Weekday[0]);  // "Monday"
console.log(Weekday[2]);  // "Wednesday"

// Step 4: Practical use case — converting a numeric value to a name.
function getDayName(day: number): string {
  return Weekday[day] ?? "Unknown";
}

console.log(getDayName(0));  // "Monday"
console.log(getDayName(4));  // "Friday"
console.log(getDayName(9));  // "Unknown"

// Step 5: The compiled JavaScript includes both mappings.
console.log(Object.keys(Weekday));
// ["0", "1", "2", "3", "4", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
```

**Expected Output:**
```
0
2
Monday
Wednesday
Monday
Friday
Unknown
[ '0', '1', '2', '3', '4', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday' ]
```

**Why This Output Occurs:** Numeric enums generate both forward and reverse mappings. The `Weekday[0]` syntax uses the reverse mapping to return `"Monday"`. The `Object.keys` output shows both the numeric keys (reverse mappings) and the string keys (forward mappings).

#### Example 2: String Enums Lack Reverse Mappings

```typescript
// Step 1: Declare a string enum.
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE",
}

// Step 2: Forward mapping works.
console.log(Color.Red);  // "RED"

// Step 3: Reverse mapping does NOT work.
console.log(Color["RED"]);  // undefined
// There is no Color["RED"] property.

// Step 4: To achieve reverse mapping, create a manual lookup.
const colorNames: Record<string, string> = {
  [Color.Red]: "Red",
  [Color.Green]: "Green",
  [Color.Blue]: "Blue",
};

console.log(colorNames[Color.Red]);  // "Red"

// Step 5: The compiled JavaScript only has forward mappings.
console.log(Object.keys(Color));  // ["Red", "Green", "Blue"]
```

**Expected Output:**
```
RED
undefined
Red
[ 'Red', 'Green', 'Blue' ]
```

**Why This Output Occurs:** String enums do not generate reverse mappings, so `Color["RED"]` returns `undefined`. The `Object.keys` output shows only the member names (forward mappings), not the values. To achieve reverse mapping for string enums, a manual lookup object must be created.

### Real-World Cases

**Case 1: Debugging and Logging**
Numeric enums with reverse mappings simplify debugging: when a numeric value appears in logs, developers can convert it to a readable name using `EnumName[value]`.

**Case 2: API Response Decoding**
APIs that return numeric codes (e.g., status codes) can be decoded into readable names using reverse mappings, improving log readability.

**Case 3: Form Validation Error Codes**
Numeric error codes with reverse mappings allow developers to display human-readable error messages by looking up the code.

---

## 5. `const enum` (Inlining Behavior)

### Definitions

**Core Definition**
A `const enum` is a TypeScript enum variant that is completely removed during compilation. Instead of generating a runtime object, the compiler inlines the enum member values directly at each use site. This produces smaller, faster code but introduces compatibility issues with certain build tools.

**Technical Definition**
`const enum` (constant enum) is declared with the `const` modifier before the `enum` keyword. During compilation, the TypeScript compiler replaces every reference to a `const enum` member with its literal value, and no JavaScript object is emitted. This is called "inlining." Because the enum is erased, it cannot be used with `isolatedModules` (used by Babel, SWC, esbuild, and some bundlers), and its values cannot be accessed at runtime. The `preserveConstEnums` compiler option emits the enum object even for `const enum` declarations, but the values are still inlined at use sites.

**Beginner-Friendly Explanation**
A `const enum` is an enum that disappears completely when your code is compiled. If you write `const enum Color { Red = "RED" }` and then use `Color.Red`, TypeScript replaces `Color.Red` with the string `"RED"` directly in your compiled code. There's no `Color` object at runtime. This makes your code smaller and faster, but it also means you can't use the enum in dynamic ways (like looping over its members). Also, some modern build tools can't handle `const enum` because they compile files one at a time and don't know the enum's values.

### Purposes

- To eliminate the runtime overhead of enum objects by inlining values at use sites.
- To reduce bundle size when enums are used extensively.
- To enable compile-time-only enums that leave no runtime trace.
- To provide a performance-optimized alternative to regular enums in performance-critical code.
- To allow the compiler to perform constant folding and dead code elimination using enum values.

### Syntax Rules and Structure

**General Syntax: `const enum` Declaration**

```typescript
const enum EnumName {
  Member1 = value1,
  Member2 = value2,
}
```

**Component Breakdown**
- `const`: The modifier that makes the enum compile-time-only.
- `enum`: The enum keyword.
- `EnumName`: The enum name.
- Members are inlined at use sites.

**General Syntax: `preserveConstEnums` Compiler Option**

```json
{
  "compilerOptions": {
    "preserveConstEnums": true
  }
}
```

**Component Breakdown**
- `preserveConstEnums`: When true, emits the enum object even for `const enum` declarations.
- Values are still inlined at use sites.

**Syntax Rules**

- `const enum` members are inlined at compile time.
- No JavaScript object is emitted for `const enum` (unless `preserveConstEnums` is enabled).
- `const enum` cannot be used with `isolatedModules: true`.
- `const enum` members must be constant expressions (no computed values).
- `const enum` cannot be accessed dynamically (e.g., no `EnumName[value]` at runtime).
- `const enum` can be used in type positions just like regular enums.

**Constraints and Limitations**

- Incompatible with `isolatedModules` (used by Babel, SWC, esbuild, and some bundlers).
- Cannot be used with dynamic access (no reverse mapping, no iteration).
- Cannot be referenced across module boundaries in some build configurations.
- Values must be compile-time constants.
- Modern bundlers may not support `const enum` without additional configuration.
- The TypeScript team recommends avoiding `const enum` in libraries and public APIs.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `const enum` Inlining

```typescript
// Step 1: Declare a const enum.
const enum Direction {
  Up = 1,
  Down = 2,
  Left = 3,
  Right = 4,
}

// Step 2: Use the enum members.
const myDirection = Direction.Up;

// Step 3: Check the compiled JavaScript.
// TypeScript compiles this to:
// const myDirection = 1;  // Inlined!
// No Direction object is emitted.

console.log(myDirection);  // 1

// Step 4: const enums cannot be accessed dynamically.
// console.log(Direction[1]);  // ❌ Error: A const enum member can only be accessed using a string literal.
// for (const d in Direction) {}  // ❌ Error: 'const' enums can only be used in property or index access expressions.
```

**Expected Output:**
```
1
```

**Why This Output Occurs:** The `const enum` member `Direction.Up` is replaced with its value `1` at compile time. The `Direction` object is not emitted, so dynamic access is not possible. This produces smaller, faster code but restricts the enum's usage.

#### Example 2: `const enum` with `isolatedModules` Error

```typescript
// Step 1: Declare a const enum in one file (enums.ts).
// const enum Status {
//   Active = "ACTIVE",
//   Inactive = "INACTIVE",
// }

// Step 2: Use it in another file (app.ts).
// import { Status } from "./enums";
// const status = Status.Active;

// Step 3: With isolatedModules enabled in tsconfig.json:
// {
//   "compilerOptions": {
//     "isolatedModules": true
//   }
// }
// TypeScript produces the error:
// ❌ Cannot access ambient const enums when 'isolatedModules' is enabled.

// Step 4: Solution — use a regular enum or a const object instead.
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
}

const status = Status.Active;
console.log(status);  // "ACTIVE"
```

**Expected Output:**
```
ACTIVE
```

**Why This Output Occurs:** With `isolatedModules` enabled, TypeScript cannot inline `const enum` values because each file is compiled independently (as Babel or SWC would do). The compiler produces an error when `const enum` is used across module boundaries. Switching to a regular enum resolves the issue at the cost of a small runtime object.

### Real-World Cases

**Case 1: Performance-Critical Code**
In performance-sensitive code where every byte and every operation matters, `const enum` can eliminate the runtime lookup overhead of regular enums.

**Case 2: Internal Libraries**
Internal libraries that are compiled with `tsc` (not Babel or SWC) can use `const enum` safely, gaining the inlining benefits without compatibility issues.

**Case 3: Avoiding `const enum` in Public APIs**
Public libraries and packages should avoid `const enum` because consumers may use different build tools that don't support it. Regular enums or `as const` objects are safer choices.

---

## 6. Enum Alternatives (Literal Unions, `as const` Objects)

### Definitions

**Core Definition**
Enum alternatives are TypeScript constructs that provide similar or superior functionality to enums without generating runtime code or introducing the compatibility issues associated with enums. The two primary alternatives are literal unions (unions of string or numeric literal types) and `as const` objects (plain objects with `as const` assertions that preserve literal types).

**Technical Definition**
Literal unions are union types composed of string or numeric literal types: `type Direction = "Up" | "Down" | "Left" | "Right"`. They exist purely in type space and produce no runtime code. `as const` objects are JavaScript objects annotated with the `as const` assertion, which makes all properties readonly and preserves their literal types. The `keyof typeof` operator combination extracts the keys of an `as const` object as a union of literal types, emulating enum member access. Both alternatives integrate better with TypeScript's structural type system than enums do, and they avoid the runtime code generation, reverse mapping, and `isolatedModules` issues associated with enums.

**Beginner-Friendly Explanation**
Enums are one way to define a set of named values, but they're not the only way—and many developers think they're not even the best way. A literal union is just a type that says "this value can be one of these strings." For example, `type Status = "active" | "inactive" | "pending"` is a literal union. An `as const` object is a regular JavaScript object marked with `as const`, which tells TypeScript to remember the exact values. Both alternatives produce less JavaScript than enums (in fact, literal unions produce no JavaScript at all), and they work better with modern build tools. Many TypeScript developers now prefer these alternatives over enums.

### Purposes

- To provide enum-like functionality without generating runtime JavaScript.
- To avoid the `isolatedModules` and `const enum` compatibility issues.
- To integrate more naturally with TypeScript's structural type system.
- To reduce bundle size by eliminating enum objects.
- To leverage `as const` for preserving literal types in object properties.

### Syntax Rules and Structure

**General Syntax: Literal Union**

```typescript
type TypeName = "value1" | "value2" | "value3";
```

**Component Breakdown**
- `type TypeName`: A type alias name.
- `"value1" | "value2" | "value3"`: A union of string literal types.

**General Syntax: `as const` Object**

```typescript
const ObjectName = {
  Member1: "value1",
  Member2: "value2",
} as const;

type TypeName = (typeof ObjectName)[keyof typeof ObjectName];
```

**Component Breakdown**
- `const ObjectName`: A const-declared object with `as const`.
- `(typeof ObjectName)`: The type of the object.
- `keyof typeof ObjectName`: The union of property names.
- `(typeof ObjectName)[keyof typeof ObjectName]`: The union of property values.

**General Syntax: Numeric Literal Union**

```typescript
type StatusCode = 200 | 404 | 500;
```

**Component Breakdown**
- A union of numeric literal types.

**Syntax Rules**

- Literal unions exist only in type space (no runtime code).
- `as const` objects exist in both type space and value space.
- `keyof typeof` extracts property names as a union of string literals.
- `(typeof Obj)[keyof typeof Obj]` extracts property values as a union.
- Literal unions are assignable from their literal values directly (no enum nominal typing).
- `as const` objects provide both named access and value unions.

**Constraints and Limitations**

- Literal unions cannot be iterated at runtime (no runtime object).
- `as const` objects can be iterated (they are real objects).
- Literal unions do not provide reverse mappings.
- `as const` objects do not provide reverse mappings (unless manually constructed).
- Literal unions are less discoverable in autocomplete in some editors (though this has improved).
- `as const` objects are more verbose than enums for simple cases.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Literal Union as Enum Alternative

```typescript
// Step 1: Define a literal union type.
type Direction = "Up" | "Down" | "Left" | "Right";

// Step 2: Use the type in a function.
function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

move("Up");     // ✅ Allowed
move("Left");   // ✅ Allowed
// move("Forward");  // ❌ Error: Argument of type '"Forward"' is not assignable to parameter of type 'Direction'.

// Step 3: Literal unions can be used with exhaustive checking.
function describeDirection(direction: Direction): string {
  switch (direction) {
    case "Up": return "Going up";
    case "Down": return "Going down";
    case "Left": return "Going left";
    case "Right": return "Going right";
    default:
      const exhaustiveCheck: never = direction;
      return exhaustiveCheck;
  }
}

console.log(describeDirection("Up"));  // "Going up"

// Step 4: No runtime code is generated for the type.
// The compiled JavaScript has no Direction object.
```

**Expected Output:**
```
Moving Up
Moving Left
Going up
```

**Why This Output Occurs:** The literal union `Direction` is purely a type-level construct. It allows only the specific string values `"Up"`, `"Down"`, `"Left"`, and `"Right"`. The exhaustive `switch` statement ensures all cases are handled, and the `never` check catches missing cases. No runtime object is generated.

#### Example 2: `as const` Object as Enum Alternative

```typescript
// Step 1: Define an as const object.
const Direction = {
  Up: "UP",
  Down: "DOWN",
  Left: "LEFT",
  Right: "RIGHT",
} as const;

// Step 2: Extract the value union type.
type Direction = (typeof Direction)[keyof typeof Direction];
// Direction = "UP" | "DOWN" | "LEFT" | "RIGHT"

// Step 3: Extract the key union type (optional).
type DirectionKey = keyof typeof Direction;
// DirectionKey = "Up" | "Down" | "Left" | "Right"

// Step 4: Use the object values.
console.log(Direction.Up);  // "UP"

// Step 5: Use the type in a function.
function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

move(Direction.Up);     // ✅ Allowed
move(Direction.Left);   // ✅ Allowed
// move("FORWARD");     // ❌ Error: Argument of type '"FORWARD"' is not assignable to parameter of type 'Direction'.

// Step 6: Iterate over the object (unlike const enum).
for (const key of Object.keys(Direction) as DirectionKey[]) {
  console.log(`${key}: ${Direction[key]}`);
}
```

**Expected Output:**
```
UP
Moving UP
Moving LEFT
Up: UP
Down: DOWN
Left: LEFT
Right: RIGHT
```

**Why This Output Occurs:** The `as const` assertion preserves the literal types of the object properties. The `(typeof Direction)[keyof typeof Direction]` type extracts the union of property values. The object can be iterated because it is a real JavaScript object, unlike a `const enum`.

### Real-World Cases

**Case 1: Modern React Development**
React components with string literal prop types (`type Variant = "primary" | "secondary"`) use literal unions instead of enums because they integrate seamlessly with JSX and produce no runtime code.

**Case 2: Redux Action Types**
Redux action types are often defined as `as const` objects or literal unions, avoiding enums' runtime overhead and compatibility issues.

**Case 3: API Configuration**
API configuration values (e.g., HTTP methods, content types) are commonly defined as `as const` objects for both type safety and runtime access.

---

## 7. Trade-offs and Modern Community Best Practices

### Definitions

**Core Definition**
The trade-offs between enums and their alternatives involve runtime code generation, type safety, bundle size, build tool compatibility, and ergonomics. Modern TypeScript community best practices generally favor literal unions and `as const` objects over enums, especially for public libraries and modern build toolchains.

**Technical Definition**
Enums generate runtime JavaScript objects (except `const enum`), which increases bundle size and introduces compatibility issues with `isolatedModules` (used by Babel, SWC, and esbuild). Enums are also one of the few TypeScript features that do not follow JavaScript's structural typing model—they are nominal, meaning two enums with the same values are not compatible. Literal unions and `as const` objects exist in type space (or both spaces for `as const`), avoid runtime overhead, and integrate with TypeScript's structural typing. The TypeScript team has acknowledged that enums have design limitations, and the community increasingly recommends alternatives.

**Beginner-Friendly Explanation**
Enums were one of TypeScript's earliest features, but they come with some baggage. They generate JavaScript code, which makes your bundle bigger. They don't work well with modern build tools like Babel and esbuild. And they're "nominal," which means two enums with the same values are treated as different types—which isn't how the rest of TypeScript works. Literal unions and `as const` objects solve these problems. They produce less code, work with all build tools, and behave like the rest of TypeScript. For these reasons, many developers now recommend avoiding enums in new code.

### Trade-off Comparison Table

| Feature | Numeric Enum | String Enum | `const enum` | Literal Union | `as const` Object |
|---------|--------------|-------------|--------------|---------------|-------------------|
| Runtime code | Yes | Yes | No (inlined) | No | Yes (object) |
| Reverse mapping | Yes | No | No | No | Manual |
| `isolatedModules` compatible | Yes | Yes | No | Yes | Yes |
| Bundle size impact | Larger | Larger | None | None | Small |
| Type safety | Lower | Higher | Higher | Highest | Highest |
| Iterable at runtime | Yes | Yes | No | No | Yes |
| Structural typing | Nominal | Nominal | Nominal | Structural | Structural |
| Autocomplete discoverability | High | High | High | Medium | High |
| Exhaustiveness checking | Yes | Yes | Yes | Yes | Yes |
| Dynamic access | Yes | No | No | No | Yes |

### Best Practices

**Prefer Literal Unions for Simple Cases**
For simple sets of string or numeric values, literal unions are the most lightweight and type-safe option. They produce no runtime code and integrate with TypeScript's structural typing.

**Use `as const` Objects for Runtime Access**
When you need runtime access to the values (iteration, dynamic lookup), `as const` objects provide the best of both worlds: type safety and runtime presence.

**Avoid Enums in Public Libraries**
Public libraries should avoid enums because consumers may use different build tools that don't support `const enum`, and regular enums increase bundle size.

**Use Regular Enums Sparingly**
Regular enums are acceptable in internal applications where the bundle size and runtime overhead are negligible and the team prefers the enum syntax.

**Avoid Heterogeneous Enums**
Heterogeneous enums are discouraged by the TypeScript team and most style guides. Use a different construct if mixed types are needed.

**Avoid `const enum` with Modern Build Tools**
`const enum` is incompatible with `isolatedModules`, which is enabled by Babel, SWC, esbuild, and many modern bundlers. Use literal unions or `as const` objects instead.

**Enable `isolatedModules` for Compatibility**
Setting `isolatedModules: true` in `tsconfig.json` ensures compatibility with single-file transpilers and catches `const enum` misuse early.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Migrating from Enum to Literal Union

```typescript
// Step 1: Original enum-based code.
enum OldStatus {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
  Pending = "PENDING",
}

function oldProcess(status: OldStatus): void {
  console.log(`Status: ${status}`);
}

oldProcess(OldStatus.Active);  // "ACTIVE"

// Step 2: Migrated to literal union.
type NewStatus = "ACTIVE" | "INACTIVE" | "PENDING";

function newProcess(status: NewStatus): void {
  console.log(`Status: ${status}`);
}

newProcess("ACTIVE");  // ✅ Allowed — no enum object needed
// newProcess("UNKNOWN");  // ❌ Error: Argument of type '"UNKNOWN"' is not assignable to parameter of type 'NewStatus'.

// Step 3: Migrated to as const object (if runtime access is needed).
const Status = {
  Active: "ACTIVE",
  Inactive: "INACTIVE",
  Pending: "PENDING",
} as const;

type StatusType = (typeof Status)[keyof typeof Status];

function modernProcess(status: StatusType): void {
  console.log(`Status: ${status}`);
}

modernProcess(Status.Active);  // "ACTIVE"
```

**Expected Output:**
```
Status: ACTIVE
Status: ACTIVE
Status: ACTIVE
```

**Why This Output Occurs:** The literal union and `as const` object both provide type-safe alternatives to the enum. The literal union produces no runtime code, while the `as const` object provides runtime access to the values.

### Real-World Cases

**Case 1: Open-Source Libraries**
Major open-source libraries (e.g., React ecosystem, Vue, Prisma) increasingly use literal unions and `as const` objects instead of enums to avoid compatibility issues and reduce bundle size.

**Case 2: Modern Build Pipelines**
Projects using Vite, esbuild, SWC, or Babel benefit from avoiding enums (especially `const enum`) because these tools compile files in isolation and cannot inline enum values.

**Case 3: Type-Safe API Design**
APIs that return string or numeric codes use literal unions or `as const` objects to define the allowed values, providing type safety without the nominal typing of enums.

---

## References

- TypeScript Handbook: Enums — https://www.typescriptlang.org/docs/handbook/enums.html
- TypeScript Handbook: Everyday Types (Literal Types) — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types
- TypeScript 5.0 Release Notes (Union Enums) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html
- TypeScript 2.4 Release Notes (String Enums) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-4.html
- TypeScript `isolatedModules` Documentation — https://www.typescriptlang.org/tsconfig#isolatedModules
- TypeScript `preserveConstEnums` Documentation — https://www.typescriptlang.org/tsconfig#preserveConstEnums
- TypeScript Wiki: FAQ — Why are enums not recommended? — https://github.com/microsoft/TypeScript/wiki/FAQ
- Total TypeScript: Enums are a Bad Idea — https://www.totaltypescript.com/why-i-dont-like-typescript-enums
- Effective TypeScript: Item 53 — Prefer Literal Types to Enums
- TypeScript ESLint: no-enum Rule — https://typescript-eslint.io/rules/no-enum/
- MDN: JavaScript Enums (no native enum) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object
- TypeScript Playground: Enums — https://www.typescriptlang.org/play/typescript/enums.ts.html
- Babel: TypeScript Enums Support — https://babeljs.io/docs/babel-plugin-transform-typescript
- SWC: TypeScript Enums — https://swc.rs/docs/configuration/compilation#jscparser