# JavaScript Conditional Statements: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
JavaScript conditional statements are control-flow constructs that execute different code blocks depending on whether a specified condition evaluates to `true` or `false`.

**Technical Definition**
Per the ECMAScript Language Specification, the `if` statement evaluates an `Expression`, applies the `ToBoolean` abstract operation to its result, and executes the first `Statement` when the result is `true`, otherwise the optional `else` `Statement`. The `switch` statement evaluates an expression, performs strict-equality comparisons against `case` clauses, and executes statements from the first matching clause until a `break` is encountered or the statement ends. The conditional (ternary) operator `?:` is the only JavaScript operator that takes three operands and selects one of two expressions based on a condition.

**Beginner-Friendly Explanation**
Conditional statements are like a fork in the road for your program. You give the computer a test—say, "Is the user logged in?"—and based on the answer, it takes one path or another. Without them, a program would run the same sequence of instructions every time, unable to react to different inputs or situations.

### Key Characteristics

- **Boolean coercion**: Conditions are not required to be Boolean; JavaScript coerces values via `ToBoolean`.
- **Branching**: They create multiple possible execution paths.
- **Composability**: Conditions can be nested, combined with logical operators, and mixed across constructs.
- **Strict matching in `switch`**: `switch` uses `===` (strict equality), not `==`.
- **Fall-through behaviour**: In `switch`, execution continues into subsequent `case` clauses unless `break` is used.
- **Expression vs statement**: `if`/`switch` are statements; `?:` is an expression that yields a value.

### Prerequisites

- Variables and data types (`number`, `string`, `boolean`, `null`, `undefined`, `object`, `bigint`).
- Comparison operators (`===`, `!==`, `>`, `<`, `>=`, `<=`).
- Logical operators (`&&`, `||`, `!`).
- Basic understanding of `true` and `false` literals.
- Familiarity with code blocks `{ }` and statement termination.

### Related Programming Areas

- Control flow and iteration (loops, `break`, `continue`).
- Error handling (`try…catch`, guard clauses).
- State machines and reducers (React, Redux).
- Input validation and form logic.
- Algorithm design (branch-and-bound, decision trees).
- Functional programming (conditional expressions, pattern matching proposals).

### Core Concepts / Features

| Concept | Category |
|---|---|
| `if` | Statement |
| `else if` | Statement chain |
| `else` | Statement fallback |
| Nested conditionals | Composition |
| Truthy and falsy | Coercion semantics |
| Conditional expressions | Expression-level logic |
| Ternary operator (`?:`) | Expression operator |
| `switch` statement | Statement |
| `case` clause | `switch` component |
| `break` statement | Control transfer |
| `default` clause | `switch` fallback |
| Fall-through | `switch` behaviour |
| Switch vs if-else | Design choice |
| Pattern matching proposals | Future language feature |

---

## Core Concepts / Features

---

## 1. `if` Statement

### Definitions

**Core Definition**
The `if` statement executes a statement if a specified condition is truthy.

**Technical Definition**
The production `IfStatement : if ( Expression ) Statement` is evaluated by: (1) evaluating `Expression` to obtain `exprRef`; (2) calling `GetValue(exprRef)`; (3) applying `ToBoolean`; (4) if the result is `true`, evaluating the first `Statement` and returning its completion; otherwise returning `undefined`.

**Beginner-Friendly Explanation**
`if` is the simplest decision-maker: "If this is true, do this." It is the building block of all conditional logic.

### Purposes

- To execute a block of code only when a condition is met.
- To guard against invalid states before proceeding.
- To introduce branching logic into an otherwise linear script.
- To act as the primary building block for more complex conditional chains.

### Syntax Rules and Structure

**Complete General Syntax**
```js
if (condition)
  statement
```

With a block statement:
```js
if (condition) {
  statements
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| `if` | Keyword that introduces the conditional statement. |
| `(` `)` | Required parentheses enclosing the condition expression. |
| `condition` | Any expression; its truthiness determines execution. |
| `statement` | A single statement or a block `{ … }` executed when the condition is truthy. |

**Syntax Rules**

- Parentheses around the condition are mandatory.
- The condition may be any expression, including assignments, function calls, and comparisons.
- A single statement may follow without braces, but braces are strongly recommended.
- The `if` statement itself has no return value.

**Constraints and Limitations**

- Without braces, the "dangling else" problem can cause an `else` to bind to an unintended `if`.
- Conditions are coerced to Boolean; values such as `0`, `""`, `null`, `undefined`, and `NaN` are falsy.

### Annotated Code Examples

**Example 1: Basic `if`**

```js
// Step 1: Declare a variable representing temperature.
const temperature = 30;

// Step 2: Evaluate the condition inside parentheses.
//        30 > 25 is true, so the block executes.
if (temperature > 25) {
  // Step 3: Log a message because the condition was truthy.
  console.log("It is a hot day.");
}
```

Expected Output:
```
It is a hot day.
```

**Breakdown**: The expression `temperature > 25` evaluates to `true`. `ToBoolean(true)` is `true`, so the block statement runs and prints the message.

**Example 2: `if` with a falsy condition**

```js
// Step 1: Declare a variable with an empty string (falsy).
let username = "";

// Step 2: The condition "" is coerced to false.
if (username) {
  console.log(`Welcome, ${username}`);
}

// Step 3: Because the condition was falsy, nothing is logged.
console.log("Done checking.");
```

Expected Output:
```
Done checking.
```

**Breakdown**: The empty string `""` is falsy. `ToBoolean("")` is `false`, so the `if` block is skipped. The subsequent `console.log` runs unconditionally.

### Real-World Cases

- **Authentication guard**: `if (!currentUser) { redirectToLogin(); }`
- **Feature flags**: `if (features.newDashboard) { renderNewDashboard(); }`
- **Input validation**: `if (email.includes("@")) { submitForm(); }`
- **Early exit**: `if (error) return handleError(error);`

### References

- MDN Web Docs — `if...else` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else
- ECMAScript Language Specification — The `if` Statement — https://tc39.es/ecma262/#sec-if-statement

---

## 2. `else if`

### Definitions

**Core Definition**
`else if` is not a distinct keyword but a conventional nesting of an `if` statement inside an `else` clause, enabling multiple mutually exclusive conditions.

**Technical Definition**
JavaScript does not have an `elseif` keyword. The pattern `else if (condition2) statement2` is parsed as `else` followed by an `IfStatement`.

**Beginner-Friendly Explanation**
`else if` lets you chain several tests together: "If the first thing is true, do this; otherwise, if the second thing is true, do that; otherwise, …"

### Purposes

- To test multiple conditions in sequence, stopping at the first truthy one.
- To implement classification or grading logic (e.g., A/B/C/D/F).
- To avoid deeply nested `if` blocks inside `else` clauses.
- To express mutually exclusive outcomes clearly.

### Syntax Rules and Structure

**Complete General Syntax**
```js
if (condition1)
  statement1
else if (condition2)
  statement2
else if (condition3)
  statement3
else
  statementN
```

**Component Breakdown**

| Component | Description |
|---|---|
| `else if` | Syntactic sugar for `else` + nested `if`. |
| `conditionN` | Each condition is evaluated only if all previous conditions were falsy. |
| `statementN` | The first matching branch executes; subsequent branches are skipped. |

**Syntax Rules**

- There is no `elseif` keyword; it must be written as two words.
- Conditions are evaluated top-to-bottom; the first truthy condition wins.
- Each `else if` must be preceded by an `if` (or another `else if`).
- The final `else` is optional.

**Constraints and Limitations**

- Long chains reduce readability; consider `switch` or a lookup table.
- Each condition expression is re-evaluated independently; avoid expensive calls in conditions.
- There is no fall-through between `else if` branches (unlike `switch` without `break`).

### Annotated Code Examples

**Example 1: Grade classification**

```js
// Step 1: Define a numeric score.
const score = 87;

// Step 2: Evaluate conditions in order.
//         87 >= 90 is false → move to next.
//         87 >= 80 is true  → execute this branch and stop.
if (score >= 90) {
  console.log("Grade: A");
} else if (score >= 80) {
  console.log("Grade: B");
} else if (score >= 70) {
  console.log("Grade: C");
} else {
  console.log("Grade: F");
}

// Expected Output:
// Grade: B
```

**Breakdown**: The first condition is false, the second is true, so `"Grade: B"` is printed. The remaining `else if` and `else` are never evaluated.

**Example 2: Multiple conditions with no final `else`**

```js
// Step 1: Declare a value that is neither positive nor negative.
const value = 0;

// Step 2: Check positivity, then negativity.
if (value > 0) {
  console.log("Positive");
} else if (value < 0) {
  console.log("Negative");
}

// Step 3: No else branch → nothing is logged for 0.
console.log("Finished.");

// Expected Output:
// Finished.
```

**Breakdown**: `value > 0` is false; `value < 0` is false. No branch executes. The final log runs unconditionally.

### Real-World Cases

- **Pricing tiers**: `if (units < 10) price = 5; else if (units < 100) price = 4; …`
- **HTTP status handling**: `if (status === 200) … else if (status === 404) …`
- **Role-based access**: `if (role === "admin") … else if (role === "editor") …`
- **Time-of-day greetings**: `if (hour < 12) "Good morning"; else if (hour < 18) "Good afternoon"; …`

### References

- MDN Web Docs — `if...else` (Description) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else
- ECMAScript Language Specification — The `if` Statement — https://tc39.es/ecma262/#sec-if-statement

---

## 3. `else`

### Definitions

**Core Definition**
The `else` clause provides a fallback statement that executes when the associated `if` condition is falsy.

**Technical Definition**
The production `IfStatement : if ( Expression ) Statement else Statement` evaluates the first `Statement` when `ToBoolean(GetValue(Expression))` is `true`; otherwise it evaluates the second `Statement`.

**Beginner-Friendly Explanation**
`else` means "otherwise"—it is what happens when the `if` test fails.

### Purposes

- To provide a default action when a condition is not met.
- To ensure that at least one branch of a decision always executes.
- To handle "none of the above" cases in a chain.
- To make the two-outcome nature of a condition explicit.

### Syntax Rules and Structure

**Complete General Syntax**
```js
if (condition)
  statement1
else
  statement2
```

**Component Breakdown**

| Component | Description |
|---|---|
| `else` | Keyword introducing the fallback branch. |
| `statement2` | Executed when `condition` is falsy. |

**Syntax Rules**

- `else` must immediately follow the `if` statement (or an `else if` chain).
- No condition is placed after `else`.
- `else` is optional; an `if` without `else` simply does nothing when the condition is falsy.
- A block `{ … }` is recommended for multiple statements.

**Constraints and Limitations**

- Only one `else` per `if` chain.
- The "dangling else" ambiguity: without braces, `else` binds to the nearest unmatched `if`.

### Annotated Code Examples

**Example 1: Pass/fail decision**

```js
// Step 1: Declare a score.
const score = 58;

// Step 2: Test the condition.
//         58 >= 60 is false → execute the else branch.
if (score >= 60) {
  console.log("Pass");
} else {
  console.log("Fail");
}

// Expected Output:
// Fail
```

**Breakdown**: The condition is falsy, so the `else` block executes.

**Example 2: Avoiding the dangling `else`**

```js
// Step 1: Define a function with nested ifs and no braces.
function checkValue(a, b) {
  if (a === 1)
    if (b === 2)
      console.log("a is 1 and b is 2");
  else
    console.log("a is not 1");
}

// Step 2: Call with a=1, b=3.
checkValue(1, 3);

// Expected Output:
// a is not 1
```

**Breakdown**: The `else` binds to the inner `if (b === 2)`, not the outer `if (a === 1)`. This is the classic dangling-else problem. Using braces avoids this ambiguity.

### Real-World Cases

- **Login fallback**: `if (user) { showDashboard(); } else { showLogin(); }`
- **Discount logic**: `if (isMember) { applyDiscount(); } else { fullPrice(); }`
- **File existence**: `if (fileExists) { read(); } else { create(); }`

### References

- MDN Web Docs — `if...else` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else
- ECMAScript Language Specification — The `if` Statement — https://tc39.es/ecma262/#sec-if-statement

---

## 4. Nested Conditionals

### Definitions

**Core Definition**
Nested conditionals are `if` (or `switch`) statements placed inside the body of another conditional statement.

**Technical Definition**
Since any `Statement` may appear in the body of an `if`, `Statement` productions can recursively contain further `IfStatement` productions, enabling arbitrary depth of conditional logic.

**Beginner-Friendly Explanation**
Nesting is like asking a follow-up question after getting an answer: "If you're an adult, are you also a member?"

### Purposes

- To express hierarchical or multi-level decision logic.
- To refine a decision based on additional criteria.
- To implement matrix-like classification (e.g., type × status).
- To model state transitions with guards.

### Syntax Rules and Structure

**Complete General Syntax**
```js
if (condition1) {
  if (condition2) {
    statementA
  } else {
    statementB
  }
} else {
  statementC
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| Outer `if` | First-level decision. |
| Inner `if` | Second-level decision evaluated only when outer condition is truthy. |
| Braces | Essential for clarity and to avoid dangling-else bugs. |

**Syntax Rules**

- Any statement can appear inside an `if` block, including another `if`.
- Depth is theoretically unlimited but practically constrained by readability.
- Always use braces when nesting to prevent ambiguous `else` binding.

**Constraints and Limitations**

- Deep nesting (>3 levels) reduces readability and maintainability.
- Early returns, logical operators, or `switch` can flatten nested logic.
- Each level adds cognitive load and potential for logic errors.

### Annotated Code Examples

**Example 1: Two-level nesting**

```js
// Step 1: Define user properties.
const isLoggedIn = true;
const isAdmin = false;

// Step 2: Outer condition checks login status.
if (isLoggedIn) {
  // Step 3: Inner condition checks role.
  if (isAdmin) {
    console.log("Admin dashboard");
  } else {
    console.log("User dashboard");
  }
} else {
  console.log("Please log in");
}

// Expected Output:
// User dashboard
```

**Breakdown**: `isLoggedIn` is true, so the outer block runs. `isAdmin` is false, so the inner `else` runs.

**Example 2: Flattening with early return (best practice)**

```js
// Step 1: Define a function with early returns to avoid nesting.
function getDashboard(user) {
  if (!user) return "Please log in";
  if (!user.isLoggedIn) return "Please log in";
  if (user.isAdmin) return "Admin dashboard";
  return "User dashboard";
}

// Step 2: Test with a logged-in non-admin.
console.log(getDashboard({ isLoggedIn: true, isAdmin: false }));

// Expected Output:
// User dashboard
```

**Breakdown**: Early returns reduce nesting depth and make the logic read top-to-bottom.

### Real-World Cases

- **E-commerce checkout**: Check cart non-empty → check shipping address → check payment method.
- **Form validation**: Check required fields → check format → check business rules.
- **Game logic**: Check collision → check health → check win condition.

### References

- MDN Web Docs — `if...else` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else
- ESLint Plugin Conditional Best Practices — https://www.npmjs.com/package/@andreasnicolaou/eslint-plugin-conditional-best-practices

---

## 5. Truthy and Falsy

### Definitions

**Core Definition**
A **truthy** value is one that evaluates to `true` in a Boolean context; a **falsy** value evaluates to `false`.

**Technical Definition**
JavaScript applies the `ToBoolean` abstract operation in Boolean contexts. The complete set of falsy values is: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`, and `document.all` (legacy). All other values are truthy.

**Beginner-Friendly Explanation**
Some values "feel" false to JavaScript even if they aren't the literal `false`. For example, `0`, `""` (empty string), and `null` all behave like `false` in an `if` test. Everything else—including `"0"`, `[]`, and `{}`—behaves like `true`.

### Purposes

- To simplify conditional checks without explicit `=== true` or `=== false`.
- To handle optional values (`null`, `undefined`) concisely.
- To enable idiomatic JavaScript patterns like `if (value)` and `value || defaultValue`.
- To understand type coercion in Boolean contexts.

### Syntax Rules and Structure

**General Syntax (implicit coercion)**
```js
if (value) { … }        // value is coerced via ToBoolean
value ? exprA : exprB   // conditional operator uses ToBoolean
```

**Component Breakdown**

| Falsy Value | Type | Description |
|---|---|---|
| `false` | Boolean | Literal false |
| `0` | Number | Zero (including `0.0`, `0x0`) |
| `-0` | Number | Negative zero |
| `0n` | BigInt | BigInt zero |
| `""` | String | Empty string |
| `null` | Null | Absence of value |
| `undefined` | Undefined | Primitive undefined |
| `NaN` | Number | Not-a-Number |
| `document.all` | Object | Legacy, implementation-defined |

**Syntax Rules**

- Truthiness is determined by `ToBoolean`, not by loose equality.
- `"0"`, `"false"`, `[]`, `{}`, `function(){}` are all truthy.
- `document.all` is the only falsy object; its behaviour is legacy and should not be relied upon.

**Constraints and Limitations**

- Implicit coercion can hide bugs (e.g., `0` being treated as "no value").
- Use explicit checks (`=== 0`, `!== undefined`) when falsy coercion is inappropriate.
- `document.all` is a deprecated, implementation-dependent falsy object.

### Annotated Code Examples

**Example 1: Falsy values bypass the `if` block**

```js
// Step 1: Array of known falsy values.
const falsyValues = [false, null, undefined, 0, -0, 0n, NaN, ""];

// Step 2: Loop through and test each.
for (const value of falsyValues) {
  if (value) {
    console.log(`${String(value)} is truthy`);
  } else {
    console.log(`${String(value)} is falsy`);
  }
}

// Expected Output:
// false is falsy
// null is falsy
// undefined is falsy
// 0 is falsy
// 0 is falsy
// 0 is falsy
// NaN is falsy
// is falsy
```

**Breakdown**: Each value is coerced via `ToBoolean`. All are in the falsy set, so each executes the `else` branch. Note `String(0n)` is `"0"` and `String("")` is `""`.

**Example 2: Truthy values that surprise beginners**

```js
// Step 1: Values that look "empty" but are truthy.
const truthySurprises = ["0", "false", [], {}, function () {}];

// Step 2: Test each.
truthySurprises.forEach((value) => {
  if (value) {
    console.log(`${JSON.stringify(value)} is truthy`);
  } else {
    console.log(`${JSON.stringify(value)} is falsy`);
  }
});

// Expected Output:
// "0" is truthy
// "false" is truthy
// [] is truthy
// {} is truthy
// undefined is truthy
```

**Breakdown**: Non-empty strings, arrays (even empty), objects (even empty), and functions are truthy. The function is serialised as `undefined` by `JSON.stringify`, but the function itself is truthy.

### Real-World Cases

- **Default parameters**: `function greet(name) { name = name || "Guest"; … }`
- **Optional chaining alternatives**: `if (user && user.address) { … }`
- **Environment variables**: `const port = process.env.PORT || 3000;`
- **Nullish coalescing complement**: `value ?? defaultValue` only falls back for `null`/`undefined`, unlike `||` which catches all falsy values.

### References

- MDN Web Docs — Truthy — https://developer.mozilla.org/en-US/docs/Glossary/Truthy
- MDN Web Docs — Falsy — https://developer.mozilla.org/en-US/docs/Glossary/Falsy
- ECMAScript Language Specification — ToBoolean — https://tc39.es/ecma262/#sec-toboolean

---

## 6. Conditional Expressions

### Definitions

**Core Definition**
A conditional expression is any expression that evaluates to different values depending on a condition.

**Technical Definition**
In ECMAScript, the `ConditionalExpression` production is defined as `LogicalORExpression ? AssignmentExpression : AssignmentExpression`, and also includes the logical OR expression itself. This means the ternary operator is the primary conditional expression, but logical operators (`&&`, `||`, `??`) also produce conditional evaluation.

**Beginner-Friendly Explanation**
A conditional expression is a question that returns a value instead of running a statement. The ternary operator is the classic example, but short-circuit logic like `a && b` is also conditional.

### Purposes

- To produce a value based on a condition without writing a full `if` statement.
- To keep code concise when assigning or returning conditional values.
- To enable functional-style conditional logic.
- To combine conditions inline within larger expressions.

### Syntax Rules and Structure

**General Syntax (ternary)**
```js
condition ? expressionIfTrue : expressionIfFalse
```

**General Syntax (logical AND)**
```js
condition && expressionIfTrue
```

**General Syntax (logical OR)**
```js
condition || expressionIfFalse
```

**Component Breakdown**

| Component | Description |
|---|---|
| `condition` | Expression coerced to Boolean. |
| `expressionIfTrue` | Value returned when condition is truthy. |
| `expressionIfFalse` | Value returned when condition is falsy. |

**Syntax Rules**

- The ternary operator is right-associative and can be chained.
- Logical operators return the actual operand value, not necessarily a Boolean.
- Conditional expressions can appear anywhere an expression is expected.

**Constraints and Limitations**

- Nested ternaries reduce readability; avoid them for complex logic.
- `&&` and `||` do not return Booleans; they return one of the operands.
- `??` only treats `null` and `undefined` as nullish, unlike `||`.

### Annotated Code Examples

**Example 1: Ternary expression assignment**

```js
// Step 1: Define age.
const age = 20;

// Step 2: Ternary produces a value.
const category = age >= 18 ? "Adult" : "Minor";

// Step 3: Log the result.
console.log(category);

// Expected Output:
// Adult
```

**Breakdown**: `age >= 18` is true, so the expression evaluates to `"Adult"`.

**Example 2: Logical AND as conditional expression**

```js
// Step 1: Define a user object.
const user = { name: "Alice", isAdmin: true };

// Step 2: Short-circuit: if user exists, access name.
const name = user && user.name;
console.log(name);

// Step 3: If user is null, short-circuit returns null.
const missing = null;
const fallback = missing && missing.name;
console.log(fallback);

// Expected Output:
// Alice
// null
```

**Breakdown**: `user && user.name` returns `"Alice"` because `user` is truthy. `missing && missing.name` returns `null` because `missing` is falsy and short-circuits.

### Real-World Cases

- **Rendering in React**: `{isLoggedIn ? <Dashboard /> : <Login />}`
- **Default values**: `const name = input || "Anonymous";`
- **Optional chaining alternative**: `const city = user && user.address && user.address.city;`
- **Configuration**: `const timeout = process.env.TIMEOUT ?? 5000;`

### References

- MDN Web Docs — Conditional (ternary) operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator
- ECMAScript Language Specification — Conditional Operator — https://tc39.es/ecma262/#sec-conditional-operator
- MDN Web Docs — Logical AND — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND

---

## 7. Ternary Operator (`?:`)

### Definitions

**Core Definition**
The ternary operator is JavaScript's only three-operand operator, used as a shorthand for `if…else` when a value must be produced.

**Technical Definition**
The `ConditionalExpression` production `ShortCircuitExpression ? AssignmentExpression : AssignmentExpression` evaluates the `ShortCircuitExpression`; if its Boolean value is `true`, it evaluates and returns the first `AssignmentExpression`; otherwise it evaluates and returns the second.

**Beginner-Friendly Explanation**
The ternary operator is a one-line `if…else` that returns a value: `condition ? valueIfTrue : valueIfFalse`.

### Purposes

- To assign a value conditionally in a single expression.
- To return different values from a function based on a condition.
- To keep inline conditionals concise.
- To avoid the verbosity of a full `if…else` block for simple choices.

### Syntax Rules and Structure

**Complete General Syntax**
```js
condition ? expressionIfTrue : expressionIfFalse
```

**Component Breakdown**

| Component | Description |
|---|---|
| `condition` | An expression whose truthiness is tested. |
| `?` | Separator introducing the true branch. |
| `expressionIfTrue` | Evaluated and returned if condition is truthy. |
| `:` | Separator introducing the false branch. |
| `expressionIfFalse` | Evaluated and returned if condition is falsy. |

**Syntax Rules**

- The condition is coerced via `ToBoolean`.
- Only one of the two expressions is evaluated (short-circuit).
- The operator is right-associative: `a ? b : c ? d : e` is `a ? b : (c ? d : e)`.
- Can be nested but readability suffers.

**Constraints and Limitations**

- Deeply nested ternaries are hard to read and maintain; prefer `if…else` or `switch` for complex logic.
- Both branches should ideally have the same type to avoid type surprises.
- Cannot contain statements, only expressions.

### Annotated Code Examples

**Example 1: Basic ternary**

```js
// Step 1: Declare a numeric age.
const age = 26;

// Step 2: Evaluate condition and choose a value.
//         age >= 21 is true → return "Beer".
const beverage = age >= 21 ? "Beer" : "Juice";

// Step 3: Log the chosen value.
console.log(beverage);

// Expected Output:
// Beer
```

**Breakdown**: The condition is true, so `"Beer"` is returned.

**Example 2: Chained ternary**

```js
// Step 1: Define a score.
const score = 75;

// Step 2: Chain ternaries for multiple ranges.
const grade =
  score >= 90 ? "A" :
  score >= 80 ? "B" :
  score >= 70 ? "C" :
  "F";

// Step 3: Log the grade.
console.log(grade);

// Expected Output:
// C
```

**Breakdown**: The operator is right-associative. `score >= 90` is false; `score >= 80` is false; `score >= 70` is true, so `"C"` is returned.

### Real-World Cases

- **UI rendering**: `return isLoading ? <Spinner /> : <Content />;`
- **API response mapping**: `const status = res.ok ? "success" : "error";`
- **CSS class selection**: `className={isActive ? "active" : "inactive"}`
- **Unit formatting**: `const unit = count === 1 ? "item" : "items";`

### References

- MDN Web Docs — Conditional (ternary) operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator
- ECMAScript Language Specification — Conditional Operator — https://tc39.es/ecma262/#sec-conditional-operator

---

## 8. `switch` Statement

### Definitions

**Core Definition**
The `switch` statement evaluates an expression and executes statements associated with the first matching `case` clause.

**Technical Definition**
A `switch` statement evaluates its `Expression`, obtains `switchValue` via `GetValue`, and compares it with each `CaseClause` expression using strict equality (`===`). Control transfers to the first matching clause and continues until a `break` or the end of the statement.

**Beginner-Friendly Explanation**
`switch` is like a multi-way `if…else if…else` that compares one value against many possible options. It is cleaner when you are checking the same variable against multiple constants.

### Purposes

- To replace long `if…else if` chains that test the same expression.
- To provide a structured, readable multi-branch selector.
- To enable fall-through grouping for shared logic.
- To improve performance in engines that optimise `switch` into jump tables.

### Syntax Rules and Structure

**Complete General Syntax**
```js
switch (expression) {
  case value1:
    statements1
    break;
  case value2:
    statements2
    break;
  default:
    statementsDefault
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| `switch` | Keyword introducing the statement. |
| `(expression)` | The value to compare against each `case`. |
| `case valueN:` | A label; if `expression === valueN`, execution starts here. |
| `break` | Optional; exits the `switch` early. |
| `default:` | Optional; executes if no `case` matches. |

**Syntax Rules**

- `switch` uses strict equality (`===`), not loose equality.
- `case` expressions are evaluated only until a match is found.
- Without `break`, execution falls through to subsequent clauses.
- Only one `default` clause is allowed; multiple `default` clauses cause a `SyntaxError`.

**Constraints and Limitations**

- Cannot directly match ranges or complex conditions; use `switch (true)` as a workaround.
- Fall-through can be a bug if `break` is forgotten.
- `case` values must be expressions, not patterns (until pattern matching is standardised).

### Annotated Code Examples

**Example 1: Day-of-week switch**

```js
// Step 1: Define a day number.
const day = 3;

// Step 2: Switch on the day value.
switch (day) {
  case 1:
    console.log("Monday");
    break;
  case 2:
    console.log("Tuesday");
    break;
  case 3:
    console.log("Wednesday");
    break; // Step 3: Exit the switch after matching.
  default:
    console.log("Another day");
}

// Expected Output:
// Wednesday
```

**Breakdown**: `day === 3` matches `case 3`. The statement `console.log("Wednesday")` runs, then `break` exits the switch, skipping `default`.

**Example 2: `switch (true)` for range matching**

```js
// Step 1: Define a score.
const score = 85;

// Step 2: Switch on true to enable boolean case expressions.
switch (true) {
  case score >= 90:
    console.log("A");
    break;
  case score >= 80:
    console.log("B");
    break;
  default:
    console.log("C or below");
}

// Expected Output:
// B
```

**Breakdown**: `switch (true)` evaluates each `case` as a Boolean expression. `score >= 90` is false; `score >= 80` is true, so `"B"` is logged.

### Real-World Cases

- **Command dispatch**: `switch (command) { case "add": …; case "remove": …; }`
- **HTTP method handling**: `switch (req.method) { case "GET": …; case "POST": …; }`
- **State machines**: `switch (state) { case "idle": …; case "loading": …; }`
- **Redux reducers**: `switch (action.type) { case "INCREMENT": …; }`

### References

- MDN Web Docs — `switch` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch
- ECMAScript Language Specification — The `switch` Statement — https://tc39.es/ecma262/#sec-switch-statement

---

## 9. `case` Clause

### Definitions

**Core Definition**
A `case` clause labels a section of code within a `switch` statement that executes when its value matches the switch expression.

**Technical Definition**
The `CaseClause : case Expression : StatementList` production evaluates `Expression`, compares it to the switch value using strict equality, and if equal, transfers control to the `StatementList`.

**Beginner-Friendly Explanation**
`case` is an option in a `switch`: "If the value equals this, start running code here."

### Purposes

- To define one possible match value in a `switch`.
- To group multiple values that share the same behaviour (via fall-through).
- To make the switch's comparison targets explicit and readable.
- To enable compile-time optimisation in engines.

### Syntax Rules and Structure

**Complete General Syntax**
```js
case value:
  statements
```

**Component Breakdown**

| Component | Description |
|---|---|
| `case` | Keyword introducing a match label. |
| `value` | Expression compared via `===` to the switch expression. |
| `:` | Separator between the label and its statements. |
| `statements` | Code executed when this case matches. |

**Syntax Rules**

- `case` must be inside a `switch` block.
- Values are compared with `===`; type matters.
- Multiple `case` labels can precede the same statement list (intentional fall-through).
- The `case` expression is only evaluated if no prior match has been found.

**Constraints and Limitations**

- Cannot use ranges or patterns directly (use `switch (true)` for ranges).
- Duplicate `case` values are allowed but only the first is reachable.
- Falling through without `break` is often unintentional.

### Annotated Code Examples

**Example 1: Multiple cases sharing logic**

```js
// Step 1: Define a fruit.
const fruit = "apple";

// Step 2: Group apple and pear together via fall-through.
switch (fruit) {
  case "apple":
  case "pear":
    console.log("Pome fruit");
    break;
  case "orange":
    console.log("Citrus fruit");
    break;
  default:
    console.log("Unknown fruit");
}

// Expected Output:
// Pome fruit
```

**Breakdown**: `case "apple"` matches; because there is no statement before `case "pear"`, execution falls through to the shared `console.log`.

**Example 2: Strict equality demonstration**

```js
// Step 1: Define a numeric value.
const value = 1;

// Step 2: Switch with a string case.
switch (value) {
  case "1":
    console.log("String one");
    break;
  case 1:
    console.log("Number one");
    break;
}

// Expected Output:
// Number one
```

**Breakdown**: `1 === "1"` is false, so the string case is skipped. `1 === 1` is true, so `"Number one"` is logged. This demonstrates strict equality.

### Real-World Cases

- **Menu selection**: `case 1: … case 2: …`
- **Error code handling**: `case 404: … case 500: …`
- **Enum-like constants**: `case STATUS.ACTIVE: …`
- **Language locale routing**: `case "en": … case "fr": …`

### References

- MDN Web Docs — `switch` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch
- ECMAScript Language Specification — The `switch` Statement — https://tc39.es/ecma262/#sec-switch-statement

---

## 10. `break` Statement

### Definitions

**Core Definition**
The `break` statement terminates the current loop or `switch` statement and transfers control to the statement following the terminated construct.

**Technical Definition**
When `break;` is encountered, the program exits the innermost enclosing `switch` or iteration statement and continues with the next statement after it. With a label, it can exit any labelled statement.

**Beginner-Friendly Explanation**
`break` is the "stop here" instruction inside a `switch` or loop. Without it, a `switch` keeps running into the next `case`.

### Purposes

- To exit a `switch` after a matching case has been handled.
- To prevent unintentional fall-through.
- To terminate a loop early when a condition is met.
- To break out of labelled blocks for complex control flow.

### Syntax Rules and Structure

**Complete General Syntax**
```js
break;
break label;
```

**Component Breakdown**

| Component | Description |
|---|---|
| `break` | Keyword that terminates the enclosing statement. |
| `label` | Optional identifier of a labelled statement to exit. |

**Syntax Rules**

- `break;` must be inside a loop or `switch` unless it has a label.
- `break label;` requires the label to be in scope.
- Cannot be used at the top level of a script, module, function, or static block.

**Constraints and Limitations**

- Only exits the innermost applicable statement (or the labelled one).
- Overuse of labels can reduce readability.

### Annotated Code Examples

**Example 1: `break` in a switch**

```js
// Step 1: Define a status code.
const status = 404;

// Step 2: Switch with break to prevent fall-through.
switch (status) {
  case 200:
    console.log("OK");
    break;
  case 404:
    console.log("Not Found");
    break; // Step 3: Exits the switch here.
  case 500:
    console.log("Server Error");
    break;
}

// Expected Output:
// Not Found
```

**Breakdown**: `case 404` matches and logs `"Not Found"`. The `break` exits the switch, so `case 500` is not evaluated.

### Real-World Cases

- **Switch handling**: Every `case` typically ends with `break` to isolate logic.
- **Search loops**: `for (const item of items) { if (item === target) { found = true; break; } }`
- **Game loops**: Break out of an animation frame loop when a condition is met.
- **Labeled blocks**: Break out of nested loops with a single label.

### References

- MDN Web Docs — `break` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/break
- ECMAScript Language Specification — The `break` Statement — https://tc39.es/ecma262/#sec-break-statement

---

## 11. `default` Clause

### Definitions

**Core Definition**
The `default` clause in a `switch` statement executes when no `case` value matches the switch expression.

**Technical Definition**
The `DefaultClause : default : StatementList` production is evaluated if no `CaseClause` matches. A `switch` statement may have at most one `default` clause.

**Beginner-Friendly Explanation**
`default` is the "else" of a `switch`: it runs when none of the cases match.

### Purposes

- To provide fallback behaviour when no case matches.
- To handle unexpected or unknown values gracefully.
- To make the switch exhaustive in intent.
- To improve robustness in input handling.

### Syntax Rules and Structure

**Complete General Syntax**
```js
switch (expression) {
  case value1:
    statements1
    break;
  default:
    statementsDefault
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| `default` | Keyword introducing the fallback clause. |
| `:` | Separator. |
| `statementsDefault` | Executed if no case matches. |

**Syntax Rules**

- Only one `default` per switch.
- `default` does not need to be last, but convention places it last.
- If no `default` and no case matches, the switch does nothing.

**Constraints and Limitations**

- Multiple `default` clauses cause a `SyntaxError`.
- `default` can fall through to subsequent cases if not the last clause and no `break` is used.

### Annotated Code Examples

**Example 1: Default fallback**

```js
// Step 1: Define an unknown command.
const command = "dance";

// Step 2: Switch with a default clause.
switch (command) {
  case "walk":
    console.log("Walking");
    break;
  case "run":
    console.log("Running");
    break;
  default:
    console.log("Unknown command");
}

// Expected Output:
// Unknown command
```

**Breakdown**: Neither case matches `"dance"`, so `default` executes.

**Example 2: `default` not last (fall-through risk)**

```js
// Step 1: Define a value.
const value = 99;

// Step 2: Default placed before a case.
switch (value) {
  default:
    console.log("Default");
    // No break → falls through to case 1.
  case 1:
    console.log("One");
    break;
}

// Expected Output:
// Default
// One
```

**Breakdown**: No case matches, so `default` runs. Without `break`, execution falls through to `case 1` and prints `"One"`. This is usually unintended.

### Real-World Cases

- **Error handling**: `default: throw new Error("Unknown action");`
- **Locale fallback**: `default: return translations.en;`
- **Command parsing**: `default: showHelp();`
- **State machines**: `default: transitionTo("idle");`

### References

- MDN Web Docs — `switch` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch
- ECMAScript Language Specification — The `switch` Statement — https://tc39.es/ecma262/#sec-switch-statement

---

## 12. Fall-through

### Definitions

**Core Definition**
Fall-through is the behaviour where a `switch` statement continues executing statements in subsequent `case` clauses after a match, until a `break` or the end of the switch.

**Technical Definition**
If `break` is omitted at the end of a `CaseClause`, execution proceeds to the next `CaseClause` (or `DefaultClause`) regardless of whether its value matches the switch expression.

**Beginner-Friendly Explanation**
Without `break`, `switch` doesn't stop after a match—it "falls through" into the next case and runs that code too.

### Purposes

- To intentionally group multiple case values that share the same logic.
- To enable "cumulative" behaviour across cases.
- To reduce code duplication when several values map to the same action.
- To act as a common pitfall that developers must guard against.

### Syntax Rules and Structure

**Complete General Syntax (intentional fall-through)**
```js
switch (expression) {
  case value1:
  case value2:
  case value3:
    sharedStatements
    break;
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| Empty cases | `case` labels with no statements between them. |
| Shared block | Statements executed for all grouped cases. |
| `break` | Prevents fall-through beyond the intended group. |

**Syntax Rules**

- Fall-through occurs automatically when `break` is omitted.
- Comment `// fall through` is a common convention to signal intentional fall-through.
- `default` can also be subject to fall-through if it is not last.

**Constraints and Limitations**

- Unintentional fall-through is a common bug source.
- Linters often flag fall-through without a comment.
- Fall-through cannot be conditionally prevented without `break` or `return`.

### Annotated Code Examples

**Example 1: Intentional fall-through for shared logic**

```js
// Step 1: Define a grade.
const grade = "B";

// Step 2: Group B and C for the same message.
switch (grade) {
  case "A":
    console.log("Excellent");
    break;
  case "B":
  case "C":
    console.log("Good");
    break;
  default:
    console.log("Needs improvement");
}

// Expected Output:
// Good
```

**Breakdown**: `case "B"` matches; no statements before `case "C"`, so execution falls through to the shared `console.log("Good")`, then `break` exits.

**Example 2: Unintentional fall-through (bug)**

```js
// Step 1: Define a direction.
const direction = "north";

// Step 2: Forgot break after north.
switch (direction) {
  case "north":
    console.log("Heading north");
  case "south":
    console.log("Heading south");
    break;
}

// Expected Output:
// Heading north
// Heading south
```

**Breakdown**: `case "north"` matches and logs `"Heading north"`. Because there is no `break`, execution falls through to `case "south"` and logs `"Heading south"` as well. This is a classic fall-through bug.

### Real-World Cases

- **Keyboard shortcuts**: Grouping `W` and `w` for the same action.
- **Error codes**: Grouping `400`, `401`, `403` as "client error".
- **Permissions**: Grouping `admin` and `owner` for elevated access.
- **Months by season**: Grouping December, January, February for "winter".

### References

- MDN Web Docs — `switch` (Breaking and fall-through) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch
- Google JavaScript Style Guide — Fall-through commented — https://google.github.io/styleguide/jsguide.html#features-switch-fall-through

---

## 13. Switch vs if-else

### Definitions

**Core Definition**
`switch` and `if…else` are both branching constructs; the choice between them depends on the nature of the condition and the number of branches.

**Technical Definition**
`switch` uses strict equality against constant case expressions and may be optimised into a jump table. `if…else` evaluates arbitrary Boolean expressions sequentially and cannot be similarly optimised.

**Beginner-Friendly Explanation**
Use `if…else` when you have different kinds of tests (ranges, complex logic). Use `switch` when you are comparing one value against many fixed options.

### Purposes

- To choose the most readable and maintainable branching construct.
- To leverage engine optimisations where applicable.
- To avoid the pitfalls of each construct (fall-through vs deep nesting).
- To align code style with team or project conventions.

### Syntax Rules and Structure

**General Syntax (if-else)**
```js
if (condition1) { … }
else if (condition2) { … }
else { … }
```

**General Syntax (switch)**
```js
switch (expression) {
  case value1: …; break;
  case value2: …; break;
  default: …;
}
```

**Component Breakdown**

| Aspect | `if…else` | `switch` |
|---|---|---|
| Condition type | Any expression | Strict equality against constants |
| Number of branches | Unlimited | Unlimited, but best for many |
| Fall-through | No | Yes (unless `break`) |
| Range support | Yes | No (without `switch (true)`) |
| Optimisation | Sequential evaluation | Potential jump table |
| Readability | Good for few complex conditions | Good for many simple matches |

**Syntax Rules**

- `switch` cannot express `>` or `<` directly.
- `if…else` cannot group multiple values without repeating conditions.
- `switch` requires `break` to isolate branches.

**Constraints and Limitations**

- `switch` cannot match objects or arrays by value (strict equality compares references).
- `if…else` chains become unwieldy beyond ~5 branches.
- Performance differences are engine-dependent and often negligible.

### Annotated Code Examples

**Example 1: Switch is clearer for many fixed values**

```js
// Step 1: Define a role.
const role = "editor";

// Step 2: Switch is concise for string constants.
switch (role) {
  case "admin":
    console.log("Full access");
    break;
  case "editor":
    console.log("Edit access");
    break;
  case "viewer":
    console.log("Read access");
    break;
  default:
    console.log("No access");
}

// Expected Output:
// Edit access
```

**Breakdown**: The switch compares `role` against three constants. `case "editor"` matches and logs `"Edit access"`.

**Example 2: if-else is necessary for ranges**

```js
// Step 1: Define a score.
const score = 85;

// Step 2: if-else handles range conditions.
if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else if (score >= 70) {
  console.log("C");
} else {
  console.log("F");
}

// Expected Output:
// B
```

**Breakdown**: `switch` cannot directly express `score >= 90`. `if…else` is the natural choice for range-based classification.

### Real-World Cases

- **Use `switch`**: Command dispatch, enum handling, Redux reducers, HTTP method routing.
- **Use `if…else`**: Range checks, complex Boolean logic, feature flags, guard clauses.
- **Use `switch (true)`**: When ranges must be combined with switch structure.

### References

- Codecademy — JavaScript Switch Case vs If-Else — https://www.codecademy.com/article/js-switch-vs-if-else
- MDN Web Docs — `switch` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch
- MDN Web Docs — `if...else` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else

---

## 14. Pattern Matching Proposals

### Definitions

**Core Definition**
Pattern matching is a proposed language feature that allows values to be tested against structural patterns and destructured in a single expression.

**Technical Definition**
The TC39 Pattern Matching proposal (Stage 1) introduces matcher patterns, a `match(){}` expression, and an `is` operator. Matcher patterns can be value patterns, structure patterns, or combinator patterns.

**Beginner-Friendly Explanation**
Pattern matching lets you write `match (value) { when { type: "user", age: > 18 } -> "Adult"; … }`, which is more declarative than nested `if` and `switch` statements.

### Purposes

- To express complex structural conditions declaratively.
- To combine testing and destructuring in one construct.
- To replace verbose nested `if` and `switch` chains.
- To bring JavaScript closer to languages like Rust and Scala.

### Syntax Rules and Structure

**Proposed General Syntax**
```js
match (subjectExpression) {
  when pattern: valueExpression;
  when pattern: valueExpression;
  default: valueExpression;
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| `match` | Keyword introducing the match expression. |
| `(subject)` | The value to match against. |
| `when` | Introduces a pattern clause. |
| `pattern` | A matcher pattern (value, structure, or combinator). |
| `:` | Separator. |
| `valueExpression` | Result if the pattern matches. |
| `default` | Fallback clause. |

**Syntax Rules (proposed)**

- Patterns can include literals, variable bindings (`let`, `const`), and logical combinators (`and`, `or`).
- The `is` operator tests a value against a pattern without a full `match`.
- Currently Stage 1; syntax may change.

**Constraints and Limitations**

- **Not standard**: Pattern matching is a proposal, not part of ECMAScript. It requires a transpiler or future runtime support.
- Stage 1 means the design is exploratory and subject to significant revision.
- Browser support is nonexistent without polyfills/transpilation.

### Annotated Code Examples

**Example 1: Proposed pattern matching syntax**

```js
// NOTE: This is a proposal and will not run in standard JavaScript.
match (res) {
  when { status: 200 or 201, let data }:
    handleSuccess(data);
  when { status: 404 }:
    handleNotFound();
  default:
    handleError(res);
}
```

**Breakdown**: The pattern `{ status: 200 or 201, let data }` matches an object with `status` 200 or 201 and binds its `data` property. This is more declarative than nested `if` checks.

**Example 2: `is` operator (proposed)**

```js
// NOTE: Proposal only.
var json = { user: ['Lily', 13] };

if (json is { user: [String and let name, Number and let age] }) {
  console.log(`User ${name} is ${age} years old.`);
}

// Expected Output (if implemented):
// User Lily is 13 years old.
```

**Breakdown**: The pattern tests that `json.user` is an array whose first element is a `String` (bound to `name`) and second element is a `Number` (bound to `age`).

### Real-World Cases (Potential)

- **Redux reducers**: Match action types and payload shapes declaratively.
- **API response handling**: Match status codes and body structures in one step.
- **Parser combinators**: Match token sequences by structure.
- **State machines**: Match state and event combinations cleanly.

### References

- TC39 Proposal: Pattern Matching — https://github.com/tc39/proposal-pattern-matching
- TC39 Pattern Matching Specification Draft — https://tc39.es/proposal-pattern-matching/
- TC39 Process Document — https://tc39.es/process-document/

---

## Summary Table of References

| Resource | URL |
|---|---|
| MDN — `if...else` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else |
| MDN — `switch` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch |
| MDN — `break` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/break |
| MDN — Truthy | https://developer.mozilla.org/en-US/docs/Glossary/Truthy |
| MDN — Falsy | https://developer.mozilla.org/en-US/docs/Glossary/Falsy |
| MDN — Conditional (ternary) operator | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator |
| ECMAScript — The `if` Statement | https://tc39.es/ecma262/#sec-if-statement |
| ECMAScript — The `switch` Statement | https://tc39.es/ecma262/#sec-switch-statement |
| ECMAScript — Conditional Operator | https://tc39.es/ecma262/#sec-conditional-operator |
| TC39 Pattern Matching Proposal | https://github.com/tc39/proposal-pattern-matching |
| Codecademy — Switch vs If-Else | https://www.codecademy.com/article/js-switch-vs-if-else |
| Google JavaScript Style Guide — Fall-through | https://google.github.io/styleguide/jsguide.html#features-switch-fall-through |