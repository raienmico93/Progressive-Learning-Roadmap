# Java Conditional Statements: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Conditional statements in Java are control flow constructs that allow a program to execute different blocks of code based on whether specified boolean expressions evaluate to `true` or `false`. They are fundamental decision-making mechanisms that enable programs to respond dynamically to varying inputs and states.

**Technical Definition:** The Java Language Specification defines the `if` statement as a construct that evaluates an expression of type `boolean` (or `Boolean`, which is unboxed) and conditionally executes a statement or block. The `if-then-else` statement provides two alternative execution paths, while the `if-then-else if` ladder extends this to multiple mutually exclusive branches. Nested conditionals occur when one `if` statement appears within the body of another, and compound conditions combine multiple boolean expressions using logical operators (`&&`, `||`, `!`, `^`).

**Beginner-Friendly Explanation:** Think of conditional statements as a program's way of making decisions—like a choose-your-own-adventure book. When the program reaches an `if` statement, it asks a yes-or-no question. If the answer is "yes" (`true`), it follows one path; if "no" (`false`), it follows another. This is how programs adapt: a thermostat checks if the temperature is below the target, a game checks if the player has enough health, and a login system checks if the password matches.

### Key Characteristics

- **Boolean-Driven:** Every condition must evaluate to a `boolean` value (`true` or `false`). Unlike some languages, Java does not permit integer-to-boolean coercion (e.g., `if (1)` is a compile-time error).
- **Block Scoping:** Statements executed conditionally are typically enclosed in braces `{ }`, creating a block with its own scope.
- **Mutual Exclusivity in Ladders:** In an `if-else if` chain, only the first matching branch executes; subsequent conditions are not evaluated.
- **Short-Circuit Evaluation:** Logical operators `&&` and `||` stop evaluating as soon as the result is determined, which can prevent errors and improve performance.
- **Dangling Else Rule:** An `else` clause binds to the nearest preceding unmatched `if` within the same block, regardless of indentation.

### Prerequisites

- Basic understanding of Java syntax: variables, data types, operators, and statements
- Familiarity with boolean expressions and comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`)
- Knowledge of logical operators (`&&`, `||`, `!`) and their truth tables
- A working Java development environment (JDK 8 or later recommended)

### Related Programming Areas

- **Control Flow:** Loops (`for`, `while`), `switch` statements, and `break`/`continue` interact closely with conditionals.
- **Boolean Logic:** Propositional logic, truth tables, De Morgan's laws, and short-circuit evaluation.
- **Program Design:** Defensive programming, input validation, and state machines rely heavily on conditionals.
- **Testing:** Branch coverage and path coverage are testing metrics directly tied to conditional logic.

## 1. The `if` Statement

### Definitions

**Core Definition:** The `if` statement executes a statement or block of statements only when a specified boolean condition evaluates to `true`.

**Technical Definition:** According to the Java Language Specification, an `if` statement consists of the keyword `if`, a parenthesized expression of type `boolean` or `Boolean` (which is unboxed), and a statement (the "then" statement). If the expression evaluates to `true`, the then-statement is executed; otherwise, control passes to the next statement following the `if`.

**Beginner-Friendly Explanation:** The `if` statement is like a gatekeeper. It checks a condition, and only if that condition is met (is `true`) does it let the code inside through. If the condition is not met, the code inside is simply skipped. For example, `if (temperature > 30)` might trigger a message saying "It's hot outside" only when the temperature actually exceeds 30 degrees.

### Purposes

- **To execute code conditionally** based on whether a specific requirement is satisfied.
- **To implement input validation** by checking whether values fall within acceptable ranges before processing.
- **To control program flow** by making execution depend on runtime state.
- **To guard against errors** by preventing operations on invalid data (e.g., checking for `null` or zero before division).

### Syntax Rules and Structure

**Complete General Syntax:**

```java
if (condition) {
    // statements to execute if condition is true
}
```

**Syntax Breakdown:**
- `if`: The keyword introducing the conditional statement.
- `(condition)`: A parenthesized expression that must evaluate to `boolean` or `Boolean`. This expression is mandatory.
- `{ }`: An optional block containing the statements to execute. If omitted, only the immediately following single statement is controlled by the `if`.

**Syntax Rules:**
- The condition expression must be of type `boolean`. Java does not allow implicit conversion from other types (e.g., `int`, `Object`) to `boolean`; `if (1)` or `if (object)` causes a compile-time error.
- A single statement does not require braces, but their use is strongly recommended to prevent logic errors when code is modified later .
- A semicolon immediately after the closing parenthesis (`if (condition);`) creates an empty statement, meaning the intended body will always execute regardless of the condition .

**Constraints and Limitations:**
- The condition is evaluated exactly once per entry into the `if` statement.
- There is no "then" keyword in Java; the statement following the condition is the implicit "then" branch.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `if` with a Single Statement**

```java
// BasicIf.java
public class BasicIf {
    public static void main(String[] args) {
        int age = 20; // Declare and initialize an integer variable
        
        // Check if age meets the voting requirement
        if (age >= 18) { // Condition: is age at least 18?
            System.out.println("You are eligible to vote."); // Executes only if true
        }
        
        System.out.println("Program continues."); // Always executes
    }
}
```

**Expected Output:**
```
You are eligible to vote.
Program continues.
```

**Why This Output Occurs:** The condition `age >= 18` evaluates to `true` because 20 is greater than or equal to 18. The block containing the print statement executes. The final `println` is outside the `if` block, so it runs unconditionally.

**Example 2: `if` with a `false` Condition**

```java
// FalseCondition.java
public class FalseCondition {
    public static void main(String[] args) {
        int temperature = 15; // Temperature in Celsius
        
        if (temperature > 30) { // Is it above 30 degrees?
            System.out.println("It's hot outside!");
        }
        
        System.out.println("Have a nice day!"); // Always runs
    }
}
```

**Expected Output:**
```
Have a nice day!
```

**Why This Output Occurs:** The condition `temperature > 30` evaluates to `false` (15 is not greater than 30). The block inside the `if` is skipped entirely. Only the statement outside the conditional executes.

### Real-World Cases

- **Form Validation:** Checking if a username field is empty before allowing submission.
- **Safety Checks:** Verifying that a divisor is not zero before performing division.
- **Feature Toggles:** Enabling or disabling functionality based on configuration flags.
- **Access Control:** Checking whether a user has administrative privileges before displaying admin options.

**Explanation:** In a banking application, an `if` statement might check `if (accountBalance >= withdrawalAmount)` before processing a withdrawal, preventing overdrafts. In a game, `if (playerHealth <= 0)` might trigger a game-over sequence.

---

## 2. The `if-else` Statement

### Definitions

**Core Definition:** The `if-else` statement provides two mutually exclusive execution paths: one for when the condition is `true` and another for when it is `false`.

**Technical Definition:** An `if-then-else` statement consists of an `if` keyword, a parenthesized boolean expression, a "then" statement, the `else` keyword, and an "else" statement. Exactly one of the two statements executes: the then-statement if the condition is `true`, or the else-statement if the condition is `false`. The two branches are never both executed.

**Beginner-Friendly Explanation:** The `if-else` statement is like a fork in the road. If the condition is true, you take the left path; if it is false, you take the right path. You never take both. For example, `if (score >= 60)` might print "Pass," and the `else` would print "Fail"—one of these two messages always prints, depending on the score.

### Purposes

- **To handle both outcomes** of a binary decision explicitly, ensuring no case is left unhandled.
- **To provide alternative behavior** when the primary condition is not satisfied.
- **To implement binary classification** such as pass/fail, valid/invalid, or on/off.
- **To ensure exhaustive handling** of a condition, making the code's logic complete.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
if (condition) {
    // statements if condition is true
} else {
    // statements if condition is false
}
```

**Syntax Breakdown:**
- `if (condition)`: The condition to evaluate.
- First block: Executes when condition is `true`.
- `else`: Keyword introducing the alternative branch.
- Second block: Executes when condition is `false`.

**Syntax Rules:**
- The `else` must immediately follow the `if` block (or the `if` statement if braces are omitted).
- No condition is specified after `else`; it serves as the catch-all for the negation of the `if` condition.
- Braces are optional for single statements but strongly recommended.

**Constraints and Limitations:**
- The `else` branch cannot exist without a preceding `if`.
- An `else` always pairs with the nearest unmatched `if` in the same block (the "dangling else" rule), which is why braces should always be used .

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Pass/Fail Decision**

```java
// PassFail.java
public class PassFail {
    public static void main(String[] args) {
        int score = 72; // Student's exam score
        
        // Determine pass or fail based on threshold
        if (score >= 60) { // Condition: is the score at least 60?
            System.out.println("Pass"); // Executes when condition is true
        } else {
            System.out.println("Fail"); // Executes when condition is false
        }
        
        System.out.println("Result processed."); // Always executes
    }
}
```

**Expected Output:**
```
Pass
Result processed.
```

**Why This Output Occurs:** The condition `score >= 60` evaluates to `true` because 72 is greater than or equal to 60. The `if` branch executes, printing "Pass." The `else` branch is skipped entirely. The final statement runs unconditionally.

**Example 2: Even or Odd**

```java
// EvenOdd.java
public class EvenOdd {
    public static void main(String[] args) {
        int number = 7; // The number to test
        
        // Use modulo operator to check divisibility by 2
        if (number % 2 == 0) { // Remainder is zero means even
            System.out.println(number + " is even.");
        } else {
            System.out.println(number + " is odd."); // Runs for non-even numbers
        }
    }
}
```

**Expected Output:**
```
7 is odd.
```

**Why This Output Occurs:** The expression `number % 2` computes the remainder when 7 is divided by 2, which is 1. Since 1 does not equal 0, the condition is `false`. The `else` branch executes, printing "7 is odd."

### Real-World Cases

- **Authentication:** `if (passwordMatches)` grants access, `else` denies it.
- **Temperature Control:** `if (temp < setpoint)` turns on heater, `else` turns it off.
- **E-Commerce:** `if (cartTotal >= freeShippingThreshold)` applies free shipping, `else` adds shipping cost.
- **Game Logic:** `if (playerHasKey)` unlocks door, `else` displays "You need a key."

**Explanation:** In a vending machine, `if (insertedAmount >= itemPrice)` dispenses the item; `else` displays "Insufficient funds." The `else` branch ensures the machine responds appropriately to both sufficient and insufficient payment.

---

## 3. The `else-if` Ladder

### Definitions

**Core Definition:** The `else-if` ladder is a chain of conditional tests where each subsequent condition is evaluated only if all previous conditions were `false`, allowing selection from multiple mutually exclusive alternatives.

**Technical Definition:** An `if-then-else if` construct consists of an initial `if` statement followed by one or more `else if` clauses, optionally terminated by a final `else`. Each `else if` provides an additional condition to test. Conditions are evaluated sequentially from top to bottom; the first condition that evaluates to `true` causes its associated block to execute, and all remaining conditions in the chain are skipped. If no condition matches, the optional final `else` block executes.

**Beginner-Friendly Explanation:** The `else-if` ladder is like a series of questions where each question is only asked if the previous ones were answered "no." For example, to grade an exam: "Is the score at least 90? No? Then is it at least 80? No? Then is it at least 70?" Once a "yes" is found, the questioning stops. This is more efficient than checking every condition independently.

### Purposes

- **To select one option from multiple possibilities** where the conditions are mutually exclusive.
- **To implement range-based classification** such as letter grades, temperature categories, or age groups.
- **To avoid deep nesting** by flattening multiple decision levels into a readable ladder.
- **To ensure only one branch executes** even when multiple conditions might theoretically be true.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
if (condition1) {
    // executes if condition1 is true
} else if (condition2) {
    // executes if condition2 is true AND condition1 was false
} else if (condition3) {
    // executes if condition3 is true AND all previous were false
} else {
    // executes if no condition matched
}
```

**Syntax Breakdown:**
- `if (condition1)`: First condition tested.
- `else if (condition2)`: Tested only if `condition1` was `false`.
- Additional `else if` clauses: Tested in order until one is `true` or the chain ends.
- `else`: Optional; provides a default when no condition matches.

**Syntax Rules:**
- Conditions are evaluated **sequentially from top to bottom**; order matters for correctness.
- Once a condition evaluates to `true`, its block executes and **all subsequent conditions are skipped** .
- The final `else` is optional; if omitted and no condition matches, nothing happens.
- Any number of `else if` clauses may be used.

**Constraints and Limitations:**
- If conditions overlap, the **first matching condition wins**, potentially producing unexpected results if the ladder is not ordered correctly.
- An `else if` must follow either an `if` or another `else if` in the same block.
- Deeply nested or very long ladders can become difficult to maintain; consider `switch` or polymorphic solutions for complex cases.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Letter Grade Assignment**

```java
// GradeCalculator.java
public class GradeCalculator {
    public static void main(String[] args) {
        int score = 85; // Exam score to evaluate
        String grade;   // Variable to hold the result
        
        // Ladder ordered from highest to lowest threshold
        if (score >= 90) {          // First: check for A
            grade = "A";
        } else if (score >= 80) {   // Only checked if score < 90
            grade = "B";
        } else if (score >= 70) {   // Only checked if score < 80
            grade = "C";
        } else if (score >= 60) {   // Only checked if score < 70
            grade = "D";
        } else {                    // Runs only if all above were false
            grade = "F";
        }
        
        System.out.println("Score: " + score + " -> Grade: " + grade);
    }
}
```

**Expected Output:**
```
Score: 85 -> Grade: B
```

**Why This Output Occurs:** The first condition `score >= 90` is `false` (85 is not ≥ 90). The second condition `score >= 80` is `true` (85 ≥ 80), so `grade` is assigned `"B"`. The remaining conditions are skipped. Note that the ladder is ordered correctly: if `score >= 70` were placed before `score >= 80`, a score of 85 would incorrectly match the lower threshold.

**Example 2: Time-of-Day Greeting**

```java
// Greeting.java
public class Greeting {
    public static void main(String[] args) {
        int hour = 14; // 24-hour format (0-23)
        
        // Determine greeting based on time of day
        if (hour < 12) {
            System.out.println("Good morning!");
        } else if (hour < 18) {      // 12:00 to 17:59
            System.out.println("Good afternoon!");
        } else if (hour < 22) {      // 18:00 to 21:59
            System.out.println("Good evening!");
        } else {                     // 22:00 to 23:59
            System.out.println("Good night!");
        }
    }
}
```

**Expected Output:**
```
Good afternoon!
```

**Why This Output Occurs:** The first condition `hour < 12` is `false` (14 is not less than 12). The second condition `hour < 18` is `true` (14 < 18), so "Good afternoon!" prints. The remaining conditions are not evaluated.

### Real-World Cases

- **Scholarship Eligibility:** Checking GPA ranges to determine award tiers.
- **Insurance Premiums:** Applying rates based on age brackets.
- **Shipping Costs:** Calculating fees based on package weight ranges.
- **User Roles:** Determining permissions based on role hierarchy.

**Explanation:** In a payroll system, an `else-if` ladder might compute tax brackets: `if (income < 10000)` → 0% tax, `else if (income < 50000)` → 15% tax, `else if (income < 100000)` → 25% tax, `else` → 35% tax. The ladder ensures each income falls into exactly one bracket.

---

## 4. Nested Conditions

### Definitions

**Core Definition:** Nested conditions occur when an `if` (or `if-else`) statement appears inside the body of another `if` or `else` block, creating hierarchical decision-making where inner conditions are evaluated only after outer conditions are satisfied.

**Technical Definition:** A nested conditional is any conditional statement that is itself the "then" statement or "else" statement of another conditional. The Java Language Specification defines this through the grammar's recursive structure: a statement can be an `if` statement, and the body of an `if` statement is itself a statement. Nesting depth is limited only by practical considerations (readability, stack depth).

**Beginner-Friendly Explanation:** Nested conditions are like a security checkpoint. First, you must pass the outer gate (e.g., "Do you have an ID?"). Only if you pass that check are you asked the inner question (e.g., "Is your ID valid?"). If you fail the first check, the second is never asked. This allows for more complex, multi-level decision logic.

### Purposes

- **To refine decision-making** by applying secondary criteria only when primary conditions are met.
- **To model real-world hierarchical logic** such as login systems (check username, then password).
- **To implement guard clauses** that validate multiple aspects of data before processing.
- **To reduce unnecessary computation** by deferring expensive inner checks until simpler outer conditions pass.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
if (outerCondition) {
    // executes if outerCondition is true
    if (innerCondition) {
        // executes if both outerCondition AND innerCondition are true
    } else {
        // executes if outerCondition is true but innerCondition is false
    }
} else {
    // executes if outerCondition is false
}
```

**Syntax Breakdown:**
- `outerCondition`: Evaluated first.
- Inner `if (innerCondition)`: Evaluated only if `outerCondition` is `true`.
- Inner `else`: Pairs with the inner `if` (the nearest unmatched `if`).
- Outer `else`: Pairs with the outer `if`.

**Syntax Rules:**
- Each `if` can have its own `else`, but the pairing follows the **nearest-if rule** unless braces disambiguate .
- Indentation is not semantically meaningful in Java; braces determine scope.
- Nested `if` statements can appear within either the `if` block, the `else` block, or both.

**Constraints and Limitations:**
- **Dangling Else Ambiguity:** Without braces, an `else` binds to the nearest unmatched `if`, which may not match the programmer's intent. Always use braces to avoid this .
- **Complexity:** Deep nesting (3+ levels) reduces readability and increases the risk of logic errors.
- **Alternative Approaches:** Sometimes `&&` (compound conditions) or early returns can flatten nested logic.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Login Validation (Nested `if-else`)**

```java
// LoginSystem.java
public class LoginSystem {
    public static void main(String[] args) {
        String username = "admin";
        String password = "secret123";
        String inputUser = "admin";
        String inputPass = "wrongpass";
        
        // Outer check: username
        if (inputUser.equals(username)) {
            // Inner check: password (only if username matched)
            if (inputPass.equals(password)) {
                System.out.println("Login successful");
            } else {
                System.out.println("Incorrect password");
            }
        } else {
            System.out.println("Incorrect username");
        }
    }
}
```

**Expected Output:**
```
Incorrect password
```

**Why This Output Occurs:** The outer condition `inputUser.equals(username)` is `true` (both are "admin"). Therefore, the inner `if-else` executes. The inner condition `inputPass.equals(password)` is `false` ("wrongpass" ≠ "secret123"), so the inner `else` runs, printing "Incorrect password." The outer `else` is skipped because the outer condition was true.

**Example 2: Nested `if` with Compound Condition Alternative**

```java
// NestedVsCompound.java
public class NestedVsCompound {
    public static void main(String[] args) {
        int age = 25;
        boolean hasLicense = true;
        
        // Nested version
        System.out.println("Nested version:");
        if (age >= 18) {
            if (hasLicense) {
                System.out.println("You can drive.");
            } else {
                System.out.println("You need a license.");
            }
        } else {
            System.out.println("You are too young.");
        }
        
        // Compound condition version (equivalent logic, flatter structure)
        System.out.println("\nCompound version:");
        if (age >= 18 && hasLicense) {
            System.out.println("You can drive.");
        } else if (age >= 18 && !hasLicense) {
            System.out.println("You need a license.");
        } else {
            System.out.println("You are too young.");
        }
    }
}
```

**Expected Output:**
```
Nested version:
You can drive.

Compound version:
You can drive.
```

**Why This Output Occurs:** In the nested version, `age >= 18` is `true`, so the inner `if (hasLicense)` is evaluated. Since `hasLicense` is `true`, "You can drive" prints. The compound version achieves the same result using `&&`: `age >= 18 && hasLicense` is `true`, so the first branch executes. Both versions produce identical output; the choice between them is a matter of readability and logical clarity.

### Real-World Cases

- **Multi-Factor Authentication:** Checking password, then a one-time code.
- **E-Commerce Checkout:** Verifying item availability, then checking payment validity.
- **Game State:** Checking if a player has a key, then if the door is locked.
- **Form Processing:** Validating that a field is non-empty, then checking its format.

**Explanation:** In a medical diagnosis system, nested conditions might first check if a patient has a fever; if so, then check for other symptoms like cough or fatigue to narrow down potential diagnoses. The inner checks are only relevant when the outer condition is met.

---

## 5. Compound Conditions

### Definitions

**Core Definition:** Compound conditions are boolean expressions formed by combining two or more simpler boolean expressions using logical operators (`&&`, `||`, `!`, `^`), allowing a single `if` statement to test multiple criteria simultaneously.

**Technical Definition:** The logical operators in Java are: `&&` (conditional AND), `||` (conditional OR), `!` (logical NOT), `^` (logical XOR), `&` (boolean AND), and `|` (boolean OR). The `&&` and `||` operators use short-circuit evaluation: the right operand is evaluated only if the result cannot be determined from the left operand alone. `&` and `|` always evaluate both operands. All logical operators have lower precedence than comparison and arithmetic operators.

**Beginner-Friendly Explanation:** Compound conditions let you ask multiple questions at once. Instead of nesting one `if` inside another, you can write `if (age >= 18 && hasLicense)`—"Is the person at least 18 AND do they have a license?" Both must be true. Or `if (isWeekend || isHoliday)`—"Is it the weekend OR a holiday?" Only one needs to be true.

### Purposes

- **To combine multiple criteria** into a single, readable condition.
- **To avoid unnecessary nesting** by expressing logical relationships directly.
- **To leverage short-circuit evaluation** for efficiency and safety (e.g., preventing null pointer exceptions).
- **To implement range checks** such as `if (x > 0 && x < 100)`.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
if (booleanExpr1 && booleanExpr2) { ... }  // AND: both must be true
if (booleanExpr1 || booleanExpr2) { ... }  // OR: at least one must be true
if (!booleanExpr) { ... }                  // NOT: inverts the value
if (booleanExpr1 ^ booleanExpr2) { ... }   // XOR: exactly one must be true
```

**Syntax Breakdown:**
- `&&`: Logical AND. Returns `true` only if both operands are `true`. Short-circuits: if left is `false`, right is not evaluated.
- `||`: Logical OR. Returns `true` if at least one operand is `true`. Short-circuits: if left is `true`, right is not evaluated.
- `!`: Logical NOT. Unary operator; inverts `true` to `false` and vice versa.
- `^`: Logical XOR. Returns `true` if operands differ; `false` if they are the same.

**Syntax Rules:**
- **Short-circuit evaluation** is the default for `&&` and `||` . This means the right operand may not be evaluated at all.
- Parentheses should be used to group expressions and override default precedence (though `&&` and `||` have lower precedence than comparisons, mixing them can be confusing without parentheses) .
- Comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) return `boolean` and are commonly used as operands.
- The bitwise operators `&` and `|` can also operate on booleans but **do not short-circuit**.

**Constraints and Limitations:**
- **Side Effects in Short-Circuit:** If the right operand has side effects (e.g., method calls that modify state), short-circuiting may skip them unexpectedly.
- **Null Safety:** Short-circuit evaluation is often used to guard against `NullPointerException`: `if (str != null && str.length() > 0)` is safe because the second condition is only evaluated when `str` is not null.
- **De Morgan's Laws:** `!(a && b)` is equivalent to `!a || !b`; `!(a || b)` is equivalent to `!a && !b`. These are useful for simplifying negated compound conditions.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Range Validation with `&&`**

```java
// RangeCheck.java
public class RangeCheck {
    public static void main(String[] args) {
        int score = 85; // Score to validate
        
        // Check if score is within the valid range 0-100
        if (score >= 0 && score <= 100) {
            System.out.println("Valid score: " + score);
        } else {
            System.out.println("Invalid score: " + score);
        }
        
        // Test with invalid value
        int badScore = 150;
        if (badScore >= 0 && badScore <= 100) {
            System.out.println("Valid score: " + badScore);
        } else {
            System.out.println("Invalid score: " + badScore);
        }
    }
}
```

**Expected Output:**
```
Valid score: 85
Invalid score: 150
```

**Why This Output Occurs:** For `score = 85`, both `85 >= 0` and `85 <= 100` are `true`, so the compound condition is `true`. For `badScore = 150`, `150 >= 0` is `true` but `150 <= 100` is `false`; since `&&` requires both to be true, the condition is `false`.

**Example 2: Short-Circuit Evaluation with `||`**

```java
// ShortCircuit.java
public class ShortCircuit {
    public static void main(String[] args) {
        String input = null;
        
        // Without short-circuit: would throw NullPointerException
        // if (input.length() > 0 || input == null) { ... } // BAD
        
        // With short-circuit: safe because second check only runs if first is false
        if (input == null || input.length() == 0) {
            System.out.println("Input is null or empty.");
        } else {
            System.out.println("Input: " + input);
        }
        
        // Demonstrate left-to-right evaluation
        int x = 5;
        boolean result = (x > 0) && (x < 10) && (x % 2 != 0);
        System.out.println("x is positive, less than 10, and odd: " + result);
    }
}
```

**Expected Output:**
```
Input is null or empty.
x is positive, less than 10, and odd: true
```

**Why This Output Occurs:** In the first condition, `input == null` is `true`. Because `||` short-circuits when the left side is `true`, `input.length() == 0` is **not evaluated**, avoiding a `NullPointerException`. In the second example, all three conditions are `true` (5 > 0, 5 < 10, 5 % 2 = 1 ≠ 0), so the compound expression evaluates to `true`.

### Real-World Cases

- **Age Verification:** `if (age >= 18 && age <= 65)` for employment eligibility.
- **Discount Eligibility:** `if (isMember || purchaseTotal > 100)` for applying promotions.
- **Date Validation:** `if (month >= 1 && month <= 12 && day >= 1 && day <= 31)`.
- **Null-Safe Navigation:** `if (user != null && user.isActive())`.
- **Exclusive Conditions:** `if (isWeekend ^ isHoliday)` for special scheduling.

**Explanation:** In a weather application, `if (temperature < 0 || windChill < -10)` might trigger a severe weather alert. In a game, `if (playerHealth <= 0 || playerLives == 0)` determines game over. Compound conditions allow expressing such rules directly without nesting.

---

## References Links

- Oracle. "Chapter 14. Blocks, Statements, and Patterns." Java Language Specification, Java SE 21 Edition. https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html
- Oracle. "Chapter 15. Expressions." Java Language Specification, Java SE 21 Edition. https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html
- Oracle. "The if-then and if-then-else Statements." The Java Tutorials. https://docs.oracle.com/javase/tutorial/java/nutsandbolts/if.html
- Runestone Academy. "If Statement Traps and Pitfalls." BHSawesome AP CSA. https://runestone.academy/ns/books/published/BHSawesome2/if-traps.html
- CS 124. "Compound Conditionals." CS 124 Course Materials, Fall 2026. https://www.cs124.org/lessons/Fall2026/java/004_compoundconditionals