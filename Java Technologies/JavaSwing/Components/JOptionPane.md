# JOptionPane — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JOptionPane` is a Swing utility class that makes it easy to pop up a standard dialog box that prompts users for a value or informs them of something. It provides static convenience methods for the most common dialog interactions: messages, confirmations, input prompts, and custom option selections.

**Technical Definition.** `public class JOptionPane extends JComponent implements Accessible`. While the `JOptionPane` class may appear complex because of the large number of methods, **almost all uses of this class are one-line calls to one of the static `showXxxDialog` methods**: `showMessageDialog`, `showConfirmDialog`, `showInputDialog`, and `showOptionDialog`. Each of these methods also comes in a `showInternalXXX` flavor that uses an internal frame (for MDI applications). All dialogs are **modal**, meaning each `showXxxDialog` method blocks the caller until the user's interaction is complete. The class is not thread-safe; all construction and access must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `JOptionPane` is the "quick dialog" tool in Swing. Instead of building a custom window for every little message, question, or input prompt, you call a single method—like `JOptionPane.showMessageDialog(...)`—and Swing handles the rest: the window, the icon, the buttons, and the positioning. It is the fastest way to display an alert, ask a yes/no question, or collect a single line of text from the user. Think of it as a set of pre-built dialog templates: "Here's a message," "Here's a question with buttons," "Here's a text box," and "Here's a custom set of buttons."

---

### Key Characteristics

1. **Static convenience methods.** Almost all usage consists of one-line calls to static methods. No instantiation is required for the most common dialog types.
2. **Four dialog families.** Message, confirm, input, and option dialogs cover the vast majority of simple interaction needs.
3. **Modal by nature.** Every `showXxxDialog` method blocks the caller until the dialog is dismissed. This is by design and is appropriate for simple interactions.
4. **Message polymorphism.** The `message` parameter is an `Object`, not just a `String`. It can be a `String`, an `Icon`, a `Component`, or an array of objects that are stacked vertically.
5. **Message type icons.** The `messageType` parameter (`ERROR_MESSAGE`, `INFORMATION_MESSAGE`, `WARNING_MESSAGE`, `QUESTION_MESSAGE`, `PLAIN_MESSAGE`) controls the look-and-feel-provided icon and layout.
6. **Option type buttons.** The `optionType` parameter (`DEFAULT_OPTION`, `YES_NO_OPTION`, `YES_NO_CANCEL_OPTION`, `OK_CANCEL_OPTION`) defines the standard button set.
7. **Internal variants.** `showInternalXxxDialog` methods use a `JInternalFrame` instead of a native window, for use inside MDI applications ( `JDesktopPane` ).
8. **EDT requirement and serialization caveat.** All methods must be called on the EDT. Serialized `JOptionPane` objects are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, and the `public static void main(String[] args)` entry point.
- Familiarity with Swing containers (`JFrame`, `JPanel`) and the EDT.
- Understanding of `SwingUtilities.invokeLater` and Swing's single-thread rule.
- Knowledge of `ActionListener` and basic event handling.
- A JDK (version 8 or later; `JOptionPane` is available in all modern JDKs).

---

### Related Programming Areas

- **`JDialog`:** The underlying window type used by `JOptionPane` dialogs.
- **`JInternalFrame`:** Used by the `showInternalXxx` variants for MDI applications.
- **`SwingUtilities.invokeLater`:** The mechanism for ensuring `JOptionPane` is called on the EDT.
- **`SwingWorker`:** Used to offload long-running tasks so the EDT can show dialogs without freezing.
- **`JFileChooser` and `JColorChooser`:** Specialised dialogs that complement `JOptionPane` for file and colour selection.

---

## Core Concepts and Features

---

## 1. JOptionPane — The Class

### Definitions

**Core Definition.** `JOptionPane` is the Swing class that provides standard dialog boxes for messages, confirmations, input, and custom options.

**Technical Definition.** `public class JOptionPane extends JComponent implements Accessible`. The class extends `JComponent` but is almost never added to a container directly; instead, its static `showXxxDialog` methods create and display a modal dialog containing a `JOptionPane` instance. The class supports extensive customisation through overloaded method variants.

**Beginner-Friendly Explanation.** `JOptionPane` is the class you call when you need to show a simple dialog. You do not create an object; you call a static method like `JOptionPane.showMessageDialog(...)`.

---

### Purposes

- To provide a quick, standardised way to display messages and collect simple input.
- To avoid the boilerplate of building custom `JDialog` windows for routine interactions.
- To ensure a consistent look and feel across all simple dialogs in an application.
- To support accessibility through standard dialog patterns.
- To offer internal-frame variants for MDI applications.

---

### Syntax Rules and Structure

#### Complete General Syntax (Method Families)

```java
// Message
JOptionPane.showMessageDialog(parentComponent, message, title, messageType);

// Confirm
int result = JOptionPane.showConfirmDialog(parentComponent, message, title, optionType);

// Input
String input = JOptionPane.showInputDialog(parentComponent, message, title, messageType);

// Option (fully custom)
int choice = JOptionPane.showOptionDialog(parentComponent, message, title,
    optionType, messageType, icon, options, initialValue);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `parentComponent` | The component that owns the dialog; `null` centres on screen. |
| `message` | The content to display; may be a `String`, `Icon`, `Component`, or `Object[]`. |
| `title` | The dialog window title. |
| `messageType` | Icon/style constant (`ERROR_MESSAGE`, `INFORMATION_MESSAGE`, etc.). |
| `optionType` | Button-set constant (`YES_NO_OPTION`, etc.). |
| `options` | Array of custom button labels or components. |
| `initialValue` | The initially focused option. |

#### Syntax Rules

- All methods are **static** and must be called on the EDT.
- `showMessageDialog` returns `void`; `showConfirmDialog` returns an `int`; `showInputDialog` returns a `String` (or `null` on cancel).
- Passing `null` for `parentComponent` centres the dialog on the screen.
- The `options` parameter, when supplied, overrides `optionType` for custom button sets.

#### Constraints and Limitations

- The static methods are modal and block the caller until dismissal.
- They are not suitable for complex, multi-component custom dialogs; use `JDialog` for those.
- In headless environments, `showXxxDialog` throws `HeadlessException`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Message Dialog

```java
import javax.swing.*;

public class MinimalMessage {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JOptionPane.showMessageDialog(                         // 1. Show message
                null,                                              // 2. No parent
                "Operation completed successfully.",               // 3. Message
                "Success",                                         // 4. Title
                JOptionPane.INFORMATION_MESSAGE);                  // 5. Icon style
        });
    }
}
```

**Expected Output.** A modal dialog appears with an information icon, the message "Operation completed successfully.", a "Success" title, and an "OK" button.

**Why the code produces this result.** `showMessageDialog` creates a dialog containing the message and an OK button. The `INFORMATION_MESSAGE` constant selects the informational icon from the current look and feel.

---

### Real-World Cases

- **Error alerts:** Displaying a message when a file cannot be saved.
- **Confirmation prompts:** Asking "Are you sure you want to delete this record?"
- **Quick input:** Collecting a single value (e.g., a name or a number) without building a form.
- **Custom choices:** Presenting a set of application-specific buttons (e.g., "Save," "Discard," "Cancel").

---

## 2. Option Pane Types — Message Dialogs

### Definitions

**Core Definition.** A message dialog is the simplest `JOptionPane` variant. It displays a message to the user and provides a single "OK" button to dismiss it.

**Technical Definition.** `showMessageDialog(Component parentComponent, Object message)` and its overloads. The method returns `void` because no user choice is captured—the user can only acknowledge the message.

**Beginner-Friendly Explanation.** A message dialog is a one-way notification: the program tells the user something, and the user clicks OK.

---

### Purposes

- To inform the user that something has happened.
- To display warnings, errors, or informational notices.
- To present a simple result that requires no user decision.
- To provide a blocking pause while the user reads the message.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JOptionPane.showMessageDialog(parent, "Message text");
JOptionPane.showMessageDialog(parent, "Message", "Title", JOptionPane.WARNING_MESSAGE);
```

#### Component Breakdown

| Overload | Parameters |
|----------|------------|
| `showMessageDialog(Component, Object)` | Parent, message. |
| `showMessageDialog(Component, Object, String, int)` | Parent, message, title, message type. |

#### Syntax Rules

- The return type is `void`; no result is captured.
- The message type controls the icon: `ERROR_MESSAGE`, `INFORMATION_MESSAGE`, `WARNING_MESSAGE`, `QUESTION_MESSAGE`, or `PLAIN_MESSAGE`.

#### Constraints and Limitations

- The user cannot provide a choice beyond acknowledging the message.
- For choices, use `showConfirmDialog` or `showOptionDialog`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Warning Message with Icon

```java
import javax.swing.*;

public class WarningMessage {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JOptionPane.showMessageDialog(                         // 1. Show
                null,
                "The file could not be saved.\nCheck disk space.", // 2. Multi-line
                "Warning",
                JOptionPane.WARNING_MESSAGE);                      // 3. Warning icon
        });
    }
}
```

**Expected Output.** A modal dialog with a warning icon, the two-line message, and an "OK" button.

**Why the code produces this result.** The `WARNING_MESSAGE` constant selects the warning icon, and the `\n` in the message creates a line break.

---

### Real-World Cases

- **Startup notifications:** "Updates are available."
- **Error reporting:** "Connection failed. Check your network."
- **Information messages:** "Your changes have been saved."

---

## 3. Option Pane Types — Confirm Dialogs

### Definitions

**Core Definition.** A confirm dialog asks the user a question and provides a set of standard option buttons (e.g., Yes/No, Yes/No/Cancel, OK/Cancel). It returns an integer indicating the user's choice.

**Technical Definition.** `showConfirmDialog(Component parentComponent, Object message, String title, int optionType)` and overloads. The return value is one of `JOptionPane.YES_OPTION`, `NO_OPTION`, `CANCEL_OPTION`, `OK_OPTION`, or `CLOSED_OPTION`.

**Beginner-Friendly Explanation.** A confirm dialog asks a question and gives the user buttons to answer it. You check which button was clicked to decide what to do.

---

### Purposes

- To ask the user for a yes/no decision.
- To confirm destructive actions (delete, overwrite, exit).
- To offer a small set of mutually exclusive choices.
- To capture a decision as an integer code for branching logic.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int result = JOptionPane.showConfirmDialog(parent, "Are you sure?");
if (result == JOptionPane.YES_OPTION) {
    // proceed
}
```

#### Component Breakdown

| Return Constant | Meaning |
|-----------------|---------|
| `YES_OPTION` | User chose Yes. |
| `NO_OPTION` | User chose No. |
| `CANCEL_OPTION` | User chose Cancel. |
| `OK_OPTION` | User chose OK. |
| `CLOSED_OPTION` | Dialog was closed without a choice. |

#### Syntax Rules

- The `optionType` determines which buttons appear (`YES_NO_OPTION`, `YES_NO_CANCEL_OPTION`, `OK_CANCEL_OPTION`, `DEFAULT_OPTION`).
- Always handle `CLOSED_OPTION` to avoid acting on a dismissed dialog.

#### Constraints and Limitations

- The button labels are look-and-feel-dependent; for custom labels, use `showOptionDialog`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Confirmation with Yes/No

```java
import javax.swing.*;

public class ConfirmDialog {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            int choice = JOptionPane.showConfirmDialog(            // 1. Show
                null,
                "Delete this record permanently?",                 // 2. Question
                "Confirm Delete",                                  // 3. Title
                JOptionPane.YES_NO_OPTION);                        // 4. Yes/No

            if (choice == JOptionPane.YES_OPTION) {                // 5. Branch
                System.out.println("Record deleted.");
            } else {
                System.out.println("Deletion cancelled.");
            }
        });
    }
}
```

**Expected Output.** A dialog with "Yes" and "No" buttons. Clicking "Yes" prints "Record deleted."; clicking "No" or closing the dialog prints "Deletion cancelled.".

**Why the code produces this result.** `showConfirmDialog` returns `YES_OPTION` or `NO_OPTION` (or `CLOSED_OPTION`), which is compared against the constants.

---

### Real-World Cases

- **Delete confirmations:** "Are you sure you want to delete this item?"
- **Exit confirmations:** "Unsaved changes will be lost. Exit anyway?"
- **Overwrite prompts:** "File already exists. Overwrite?"

---

## 4. Option Pane Types — Input Dialogs

### Definitions

**Core Definition.** An input dialog prompts the user to enter a value. It returns the entered value as a `String`, or `null` if the user cancelled.

**Technical Definition.** `showInputDialog(Component parentComponent, Object message)` and overloads. The method returns a `String` containing the user's input, or `null` if the dialog was cancelled. An overload accepts an `Object[]` selection values array, presenting a combo box or list for the user to choose from.

**Beginner-Friendly Explanation.** An input dialog asks the user to type something or pick something from a list, and gives you back what they entered.

---

### Purposes

- To collect a single line of text from the user.
- To let the user choose from a predefined list of values.
- To prompt for a simple value without building a full form.
- To return `null` on cancellation for clean branching.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
String input = JOptionPane.showInputDialog(parent, "Enter your name:");
if (input != null) {
    // use input
}
```

#### Component Breakdown

| Return Value | Meaning |
|--------------|---------|
| `String` | The user's input. |
| `null` | The user cancelled the dialog. |

#### Syntax Rules

- Always check for `null` before using the returned `String`.
- For a selection list, pass an `Object[]` of choices; the user picks one and its `toString()` is returned.
- The input is always a `String`; parse it if numeric input is needed.

#### Constraints and Limitations

- No built-in validation; the programmer must validate the returned string.
- For passwords or sensitive input, use `JPasswordField` inside a custom `JDialog` instead.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Text Input with Null Check

```java
import javax.swing.*;

public class InputDialog {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            String name = JOptionPane.showInputDialog(             // 1. Show input
                null,
                "Please enter your name:",
                "Name Input",
                JOptionPane.QUESTION_MESSAGE);                     // 2. Question icon

            if (name != null && !name.trim().isEmpty()) {          // 3. Null check
                System.out.println("Hello, " + name + "!");
            } else {
                System.out.println("No name entered.");
            }
        });
    }
}
```

**Expected Output.** A dialog with a text field and "OK"/"Cancel" buttons. Entering a name and clicking OK prints a greeting; cancelling or entering nothing prints "No name entered."

**Why the code produces this result.** `showInputDialog` returns the entered text as a `String`, or `null` on cancel. The null and empty checks prevent errors.

---

### Real-World Cases

- **Name prompts:** "Enter your username."
- **Numeric input:** "How many copies?" (parsed with `Integer.parseInt`).
- **Selection from a list:** Choosing a department or category from a combo box.

---

## 5. Option Pane Types — Option Dialogs

### Definitions

**Core Definition.** `showOptionDialog` is the "Grand Unification" of the other three dialog types. It provides full control over the message, icon, button set, and initial focus, and returns the index of the chosen option.

**Technical Definition.** `showOptionDialog(Component parentComponent, Object message, String title, int optionType, int messageType, Icon icon, Object[] options, Object initialValue)`. The return value is the index of the selected option (0-based), or `CLOSED_OPTION` if the dialog was dismissed.

**Beginner-Friendly Explanation.** An option dialog is the fully customisable version: you can supply any set of buttons with any labels, choose the icon, and decide which option is focused initially.

---

### Purposes

- To present a custom set of buttons with application-specific labels.
- To combine the functionality of message, confirm, and input dialogs into one method.
- To control the icon and initial focus precisely.
- To return an index for branching based on the chosen option.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Object[] options = {"Save", "Discard", "Cancel"};
int choice = JOptionPane.showOptionDialog(parent, message, title,
    JOptionPane.DEFAULT_OPTION, JOptionPane.WARNING_MESSAGE,
    null, options, options[0]);
if (choice == 0) { /* Save */ }
else if (choice == 1) { /* Discard */ }
else if (choice == 2) { /* Cancel */ }
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `optionType` | Usually `DEFAULT_OPTION` when custom `options` are supplied. |
| `options` | Array of button labels (or components). |
| `initialValue` | The option that receives initial focus. |

#### Syntax Rules

- The return value is the **index** of the chosen option, not a predefined constant.
- `CLOSED_OPTION` ( -1 ) indicates the dialog was dismissed.
- Custom `options` override the standard button set.

#### Constraints and Limitations

- The index-based return requires the programmer to map indices to actions carefully.
- `CLOSED_OPTION` must be handled to avoid treating a dismissal as a button press.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Save/Discard/Cancel Dialog

```java
import javax.swing.*;

public class OptionDialog {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            Object[] options = {"Save", "Discard", "Cancel"};      // 1. Custom labels
            int choice = JOptionPane.showOptionDialog(
                null,
                "You have unsaved changes. What would you like to do?",
                "Unsaved Changes",
                JOptionPane.DEFAULT_OPTION,                        // 2. Custom options
                JOptionPane.WARNING_MESSAGE,
                null,
                options,
                options[0]);                                       // 3. Initial focus

            switch (choice) {
                case 0: System.out.println("Saving..."); break;     // 4. Index 0
                case 1: System.out.println("Discarding..."); break;
                case 2: System.out.println("Cancelled."); break;
                default: System.out.println("Dialog closed."); break;
            }
        });
    }
}
```

**Expected Output.** A warning dialog with three custom buttons: "Save," "Discard," and "Cancel." Clicking each prints the corresponding message; closing the dialog prints "Dialog closed.".

**Why the code produces this result.** `showOptionDialog` returns the index of the chosen button (0, 1, or 2) or `CLOSED_OPTION`. The switch maps the index to the appropriate action.

---

### Real-World Cases

- **Unsaved-changes prompts:** "Save / Discard / Cancel."
- **Multi-step wizards:** Choosing "Back / Next / Finish."
- **Custom survey responses:** Presenting application-specific choices.

---

## 6. Option Pane Methods — showMessageDialog()

### Definitions

**Core Definition.** `showMessageDialog` displays a message dialog with an "OK" button and returns `void`.

**Technical Definition.** `public static void showMessageDialog(Component parentComponent, Object message)` and overloads. The method blocks until the user dismisses the dialog.

**Beginner-Friendly Explanation.** `showMessageDialog` is the simplest way to tell the user something.

---

### Purposes

- To inform the user of an event or state.
- To display errors, warnings, or confirmations of completed operations.
- To provide a blocking pause while the message is read.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JOptionPane.showMessageDialog(parent, "Message", "Title", JOptionPane.PLAIN_MESSAGE);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `message` | The content to display (String, Icon, Component, or Object[]). |
| `messageType` | Icon/style constant. |

#### Syntax Rules

- Returns `void`; no result is captured.
- The message can be multi-line using `\n`.

#### Constraints and Limitations

- No user choice beyond acknowledgment.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Error Message

```java
import javax.swing.*;

public class ErrorMessage {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() ->
            JOptionPane.showMessageDialog(                        // 1. Show
                null,
                "Failed to connect to the database.",
                "Connection Error",
                JOptionPane.ERROR_MESSAGE));                       // 2. Error icon
    }
}
```

**Expected Output.** A dialog with an error icon, the message, and an "OK" button.

**Why the code produces this result.** `ERROR_MESSAGE` selects the error icon from the look and feel.

---

### Real-World Cases

- **Error alerts** for failed operations.
- **Completion notices** ("Export finished").
- **Startup messages** ("Configuration loaded").

---

## 7. Option Pane Methods — showConfirmDialog()

### Definitions

**Core Definition.** `showConfirmDialog` displays a dialog with a standard set of option buttons and returns an integer indicating the user's choice.

**Technical Definition.** `public static int showConfirmDialog(Component parentComponent, Object message, String title, int optionType)`. Returns one of the option constants (`YES_OPTION`, `NO_OPTION`, `CANCEL_OPTION`, `OK_OPTION`, `CLOSED_OPTION`).

**Beginner-Friendly Explanation.** `showConfirmDialog` asks a question and returns which button the user clicked.

---

### Purposes

- To capture a yes/no or yes/no/cancel decision.
- To confirm destructive or irreversible actions.
- To branch application logic based on the user's choice.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int result = JOptionPane.showConfirmDialog(parent, "Proceed?",
    "Confirm", JOptionPane.YES_NO_OPTION);
```

#### Component Breakdown

| Constant | Value |
|----------|-------|
| `YES_OPTION` | 0 |
| `NO_OPTION` | 1 |
| `CANCEL_OPTION` | 2 |
| `OK_OPTION` | 0 |
| `CLOSED_OPTION` | -1 |

#### Syntax Rules

- The `optionType` determines the button set.
- Always handle `CLOSED_OPTION`.

#### Constraints and Limitations

- Button labels are look-and-feel-dependent.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Yes/No/Cancel Confirmation

```java
import javax.swing.*;

public class ConfirmThreeWay {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            int choice = JOptionPane.showConfirmDialog(            // 1. Show
                null,
                "Save changes before closing?",
                "Unsaved Changes",
                JOptionPane.YES_NO_CANCEL_OPTION);                 // 2. Three buttons

            switch (choice) {                                       // 3. Branch
                case JOptionPane.YES_OPTION: System.out.println("Saving..."); break;
                case JOptionPane.NO_OPTION: System.out.println("Closing without saving."); break;
                case JOptionPane.CANCEL_OPTION: System.out.println("Returning to editor."); break;
                default: System.out.println("Dialog closed."); break;
            }
        });
    }
}
```

**Expected Output.** A dialog with "Yes," "No," and "Cancel" buttons. Each choice prints the corresponding message.

**Why the code produces this result.** The switch maps the returned constant to the appropriate action.

---

### Real-World Cases

- **Unsaved changes prompts.**
- **Delete confirmations.**
- **Navigation confirmations** ("Leave this page?").

---

## 8. Option Pane Methods — showInputDialog()

### Definitions

**Core Definition.** `showInputDialog` prompts the user for input and returns the entered value as a `String`, or `null` if cancelled.

**Technical Definition.** `public static String showInputDialog(Component parentComponent, Object message)` and overloads. An overload accepting an `Object[]` selection values displays a combo box or list for the user to choose from.

**Beginner-Friendly Explanation.** `showInputDialog` asks the user to type something and gives you the text they entered.

---

### Purposes

- To collect a single value from the user.
- To offer a choice from a predefined list.
- To return `null` on cancellation for safe branching.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
String input = JOptionPane.showInputDialog(parent, "Enter value:");
if (input != null) { /* use input */ }
```

#### Component Breakdown

| Return | Meaning |
|--------|---------|
| `String` | The user's input. |
| `null` | The user cancelled. |

#### Syntax Rules

- Always check for `null`.
- For numeric input, parse the returned string.

#### Constraints and Limitations

- No validation is performed on the returned string.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Numeric Input with Parsing

```java
import javax.swing.*;

public class NumericInput {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            String s = JOptionPane.showInputDialog(                // 1. Show input
                null, "Enter a number:", "Number Input",
                JOptionPane.QUESTION_MESSAGE);
            if (s != null) {                                        // 2. Null check
                try {
                    int n = Integer.parseInt(s);                    // 3. Parse
                    System.out.println("Square: " + (n * n));
                } catch (NumberFormatException ex) {
                    System.out.println("Invalid number.");
                }
            }
        });
    }
}
```

**Expected Output.** Entering a valid number prints its square; entering non-numeric text or cancelling prints an error message.

**Why the code produces this result.** The returned `String` is parsed with `Integer.parseInt`, and the `try-catch` handles invalid input.

---

### Real-World Cases

- **Name or email prompts.**
- **Numeric value collection** (with parsing).
- **Selection from a list** (using the `Object[]` overload).

---

## 9. Option Pane Methods — showOptionDialog()

### Definitions

**Core Definition.** `showOptionDialog` provides full control over the dialog's message, icon, button set, and initial focus, and returns the index of the chosen option.

**Technical Definition.** `public static int showOptionDialog(Component parentComponent, Object message, String title, int optionType, int messageType, Icon icon, Object[] options, Object initialValue)`. The return value is the 0-based index of the selected option, or `CLOSED_OPTION`.

**Beginner-Friendly Explanation.** `showOptionDialog` lets you build a dialog with exactly the buttons and labels you want.

---

### Purposes

- To present application-specific button choices.
- To combine message, confirmation, and input functionality.
- To control the icon and initial focus precisely.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Object[] opts = {"A", "B", "C"};
int idx = JOptionPane.showOptionDialog(parent, "Choose:", "Title",
    JOptionPane.DEFAULT_OPTION, JOptionPane.PLAIN_MESSAGE,
    null, opts, opts[0]);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `options` | Custom button labels or components. |
| `initialValue` | Initially focused option. |

#### Syntax Rules

- The return is an index, not a predefined constant.
- `CLOSED_OPTION` indicates dismissal.

#### Constraints and Limitations

- Index mapping must be handled manually.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Button Set

```java
import javax.swing.*;

public class CustomOptions {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            String[] opts = {"Approve", "Reject", "Review"};        // 1. Labels
            int idx = JOptionPane.showOptionDialog(                 // 2. Show
                null, "Select an action:", "Workflow",
                JOptionPane.DEFAULT_OPTION,
                JOptionPane.QUESTION_MESSAGE,
                null, opts, opts[0]);

            if (idx >= 0 && idx < opts.length) {                    // 3. Guard
                System.out.println("Action: " + opts[idx]);
            } else {
                System.out.println("Dialog closed.");
            }
        });
    }
}
```

**Expected Output.** A dialog with "Approve," "Reject," and "Review" buttons. Clicking each prints the action name; closing prints "Dialog closed.".

**Why the code produces this result.** The returned index is used to look up the chosen label in the `opts` array.

---

### Real-World Cases

- **Workflow approvals** (Approve/Reject/Review).
- **Custom save prompts** (Save/Discard/Cancel).
- **Multi-option surveys.**

---

## 10. JOptionPane Best Practices

### Definitions

**Core Definition.** `JOptionPane` best practices are the conventions and patterns that ensure dialogs are shown safely on the EDT, handled correctly, and used appropriately for the interaction's complexity.

**Technical Definition.** Best practices include: always calling `showXxxDialog` on the EDT via `SwingUtilities.invokeLater`; never performing long-running work on the EDT while a dialog is open; checking `null` returns from `showInputDialog` and handling `CLOSED_OPTION` from confirm/option dialogs; and choosing `JOptionPane` only for simple interactions, using `JDialog` for complex custom dialogs.

**Beginner-Friendly Explanation.** Best practices for option panes are the "rules of thumb" that prevent frozen GUIs, null-pointer crashes, and inappropriate use of the static dialog methods. They include things like "always call these methods on the GUI thread," "check for null or closed results," and "don't try to build a complex form with `JOptionPane`."

---

### Purposes

- To maintain thread safety by respecting the EDT.
- To prevent `NullPointerException` from unhandled cancel results.
- To avoid freezing the GUI by keeping long tasks off the EDT.
- To choose the right tool: `JOptionPane` for simple dialogs, `JDialog` for complex ones.
- To provide consistent, accessible dialog experiences.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {                             // 1. EDT
    int result = JOptionPane.showConfirmDialog(
        parent, "Proceed?", "Confirm", JOptionPane.YES_NO_OPTION);
    if (result == JOptionPane.YES_OPTION) {                    // 2. Check result
        // proceed
    } else if (result == JOptionPane.CLOSED_OPTION) {          // 3. Handle close
        // user dismissed
    }
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| EDT | `SwingUtilities.invokeLater(...)` around every `showXxxDialog` call. |
| Null/close checks | `if (input != null)` for input; `CLOSED_OPTION` handling for confirm/option. |
| No long tasks on EDT | Use `SwingWorker` for background work; show dialogs before/after. |
| Right tool | `JOptionPane` for simple one-shot dialogs; `JDialog` for complex forms. |
| Parent component | Pass a parent for proper centring and modality ownership. |
| No blocking I/O on EDT | Never read files or query databases inside the dialog call. |

#### Syntax Rules

- **Always call `showXxxDialog` on the EDT.** Calling from a background thread freezes or crashes the GUI.
- **Always check the result** of `showInputDialog` for `null` and confirm/option dialogs for `CLOSED_OPTION`.
- **Never perform long-running tasks on the EDT** while a dialog is open; use `SwingWorker` and show the dialog before or after the task.
- **Use `JOptionPane` only for simple interactions.** For complex multi-component dialogs, use `JDialog`.

#### Constraints and Limitations

- The static methods are modal and block the EDT; this is acceptable for quick interactions but not for long operations.
- Custom button labels in `showOptionDialog` require careful index-to-action mapping.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Confirmation with EDT and Result Handling

```java
import javax.swing.*;

public class BestPracticeConfirm {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeConfirm::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(300, 150);
        frame.setLocationRelativeTo(null);

        JButton delete = new JButton("Delete");
        delete.addActionListener(e -> {
            int result = JOptionPane.showConfirmDialog(             // 1. On EDT
                frame, "Delete permanently?", "Confirm",
                JOptionPane.YES_NO_OPTION);
            if (result == JOptionPane.YES_OPTION) {                // 2. Check
                System.out.println("Deleted.");
            } else if (result == JOptionPane.CLOSED_OPTION) {      // 3. Close
                System.out.println("Dialog dismissed.");
            }
            // NO_OPTION: do nothing
        });

        frame.add(delete);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Clicking "Delete" opens a confirmation dialog on the EDT. Clicking "Yes" prints "Deleted."; clicking "No" does nothing; closing the dialog prints "Dialog dismissed.".

**Why the code produces this result.** The dialog is invoked on the EDT via the `ActionListener` (which runs on the EDT). The result is checked for `YES_OPTION` and `CLOSED_OPTION`, with no action for `NO_OPTION`.

---

#### Example 2: Common Mistake — Calling `JOptionPane` from a Background Thread

```java
import javax.swing.*;

public class BadThreadUsage {
    public static void main(String[] args) {
        new Thread(() -> {                                         // WRONG: background
            JOptionPane.showMessageDialog(                         // 1. Off-EDT
                null, "Hello from background thread!");
        }).start();
    }
}
```

**Expected Output.** The dialog may appear, but it can behave unpredictably, freeze, or throw concurrency-related exceptions because Swing components are not thread-safe.

**Why this is wrong.** `JOptionPane` is a Swing component and must be called on the EDT. Calling it from a background thread violates Swing's threading policy and can cause intermittent, hard-to-reproduce bugs. The correct approach is to wrap the call in `SwingUtilities.invokeLater`.

---

### Real-World Cases

- **Production Swing applications** wrap all `JOptionPane` calls in `SwingUtilities.invokeLater` and handle all possible return values.
- **Task completion notifications** show dialogs in `SwingWorker.done()`, which runs on the EDT.
- **Educational examples** use the best-practice skeleton to demonstrate correct EDT usage and result handling from the start.

---

## References

Oracle. *Class JOptionPane*. Java Platform, Standard Edition 25 API Specification. https://docs.oracle.com/en/java/javase/25/docs/api/java.desktop/javax/swing/JOptionPane.html

Oracle. *How to Make Dialogs*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/dialog.html

Oracle. *Class JOptionPane*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JOptionPane.html

Oracle. *Class JOptionPane*. Java Development Kit Version 17 API Specification. https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/javax/swing/JOptionPane.html

Stack Overflow. *JOptionPane and EDT best practices*. https://stackoverflow.com/questions/... (EDT invocation, null parent, thread safety)

Stack Overflow. *Is JOptionPane.showMessageDialog thread safe?*. https://stackoverflow.com/questions/... (thread-safety discussion)

Cleverence. *Cách tạo hộp thoại trong Java: Swing JOptionPane, JDialog và JavaFX*. https://www.cleverence.com/articles/... (Vietnamese tutorial covering the four dialog types)