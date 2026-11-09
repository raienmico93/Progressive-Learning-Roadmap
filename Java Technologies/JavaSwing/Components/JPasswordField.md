# JPasswordField — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JPasswordField` is a Swing component that provides a specialized single-line text field for password entry. It is a subclass of `JTextField` that hides the characters typed by the user, displaying an alternative echo character instead.

**Technical Definition.** `public class JPasswordField extends JTextField`. `JPasswordField` is a lightweight component that allows the editing of a single line of text where the view indicates something was typed, but does not show the original characters. It is provided separately from `JTextField` to make it easier to safely change the UI for the text field without affecting password entries. The class provides a `getPassword()` method that returns the password as a `char[]` rather than a `String`, and a `setEchoChar(char)` method that controls the character displayed in place of the actual input. By default, the echo character is `'*'`. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `JPasswordField` is the box where users type their password. Unlike a regular text field, it does not show what they type—it shows asterisks (or another character) instead, so that anyone looking over their shoulder cannot read the password. When you need to check the password, you call `getPassword()`, which gives you the characters as a special array that you can erase from memory after you are done with it.

---

### Key Characteristics

1. **Character masking.** `JPasswordField` hides the actual characters typed by the user and displays an echo character (by default, `'*'`) in their place. This prevents visual exposure of sensitive credentials.
2. **`char[]`-based retrieval.** The `getPassword()` method returns the password as a `char[]` rather than a `String`. This is a deliberate security design: a `String` is immutable and remains in memory until garbage collection, whereas a `char[]` can be explicitly cleared by overwriting its elements with zeros.
3. **Deprecated `getText()`.** The inherited `getText()` method is deprecated in `JPasswordField` because it returns the password as a `String`, which cannot be cleared from memory. `getPassword()` should always be used instead.
4. **Configurable echo character.** The echo character can be changed via `setEchoChar(char)` and retrieved via `getEchoChar()`. Setting the echo character to `'\u0000'` (zero) disables masking and shows the text as typed, which is a security vulnerability if used unintentionally.
5. **Source compatibility with `java.awt.TextField`.** `JPasswordField` is intended to be source-compatible with `java.awt.TextField` used with `echoChar` set, making migration easier.
6. **Input method interaction.** By default, `JPasswordField` disables input methods because input characters might otherwise be visible while being composed. Input methods can be enabled explicitly with `enableInputMethods(true)` if needed.
7. **Single-thread rule and serialization caveat.** `JPasswordField` is not thread-safe and must be accessed on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, and the `public static void main(String[] args)` entry point.
- Familiarity with `JTextField` and its inherited API (text fields, columns, editable state, document listeners).
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `char[]` arrays and the `java.util.Arrays` utility class for clearing sensitive data.
- A JDK (version 8 or later; `JPasswordField` is available in all modern JDKs).

---

### Related Programming Areas

- **`JTextField`:** The superclass of `JPasswordField`; most of the text field API (columns, editable, enabled, selection) is inherited.
- **`Document` and `DocumentListener`:** The underlying model for observing changes to the password input.
- **`ActionListener`:** Handles the Enter key for form submission.
- **Security best practices:** Clearing `char[]` arrays, avoiding `String` for passwords, and never disabling masking in production code.
- **`JLabel` with `setLabelFor`:** Associating a password field with a descriptive label for accessibility.

---

## Core Concepts and Features

---

## 1. JPasswordField — The Class

### Definitions

**Core Definition.** `JPasswordField` is the Swing class for creating password input fields that mask the typed characters and provide secure retrieval via `char[]`.

**Technical Definition.** `public class JPasswordField extends JTextField`. The class overrides the `getText()` method (deprecating it) and introduces `getPassword()` and `setEchoChar(char)`. The underlying document model is a `PlainDocument` by default. The UI delegate renders the echo character in place of the actual text.

**Beginner-Friendly Explanation.** `JPasswordField` is the password version of a text field. You create it just like a `JTextField`, but when the user types, they see asterisks instead of letters. When you need the password, you call `getPassword()` to get a character array that you can wipe clean.

---

### Purposes

- To provide a secure text input control for password entry that hides the typed characters from visual inspection.
- To offer a `char[]`-based retrieval mechanism that allows the password to be cleared from memory after use.
- To prevent the accidental exposure of passwords through `String` objects that linger in memory.
- To serve as the standard Swing component for login forms, authentication dialogs, and any interface requiring sensitive input.
- To integrate with the broader `JTextField` API for layout, validation, and event handling.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JPasswordField field = new JPasswordField();
JPasswordField field = new JPasswordField(int columns);
JPasswordField field = new JPasswordField(String text);
JPasswordField field = new JPasswordField(String text, int columns);
JPasswordField field = new JPasswordField(Document doc, String text, int columns);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JPasswordField()` | Creates an empty password field with 0 columns. |
| `JPasswordField(int columns)` | Creates an empty field with the specified preferred width. |
| `JPasswordField(String text)` | Creates a field with the specified initial text (masked). |
| `JPasswordField(String text, int columns)` | Creates a field with the specified text and column count. |
| `JPasswordField(Document doc, String text, int columns)` | Creates a field using the given text storage model. |

#### Syntax Rules

- The default echo character is `'*'`.
- `getText()` is deprecated and should not be used; use `getPassword()` instead.
- Construction must occur on the EDT.

#### Constraints and Limitations

- `JPasswordField` is for single-line input only.
- The column count is a hint for preferred width, not a limit on characters.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JPasswordField

```java
import javax.swing.*;
import java.awt.*;
import java.util.Arrays;

public class MinimalPasswordField {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JPasswordField");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPasswordField passwordField = new JPasswordField(20); // 1. Create
            passwordField.addActionListener(e -> {                 // 2. On Enter
                char[] password = passwordField.getPassword();      // 3. Secure read
                System.out.println("Password length: " + password.length);
                Arrays.fill(password, '0');                         // 4. Clear memory
            });

            JPanel panel = new JPanel(new FlowLayout(FlowLayout.LEFT));
            panel.add(new JLabel("Password:"));
            panel.add(passwordField);

            frame.add(panel);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a “Password:” label and a 20-column password field. Typing a password displays asterisks. Pressing Enter prints the password length to the console and clears the character array from memory.

**Why the code produces this result.** The `JPasswordField` masks the input with the default echo character `'*'`. The `ActionListener` fires on Enter. `getPassword()` returns the password as a `char[]`, which is printed (its length) and then overwritten with `'0'` characters using `Arrays.fill`.

---

### Real-World Cases

- **Login dialogs:** Password fields are the standard control for entering credentials in authentication forms.
- **Registration forms:** Users choose a password and confirm it in two `JPasswordField` instances.
- **Settings panels:** Applications that connect to external services (databases, APIs) use password fields for secret keys or tokens.
- **Encryption tools:** Password-based encryption utilities use `JPasswordField` to collect the passphrase.

---

## 2. Password Field Creation

### Definitions

**Core Definition.** Password field creation is the process of instantiating a `JPasswordField` with the desired column width and optional initial content.

**Technical Definition.** `JPasswordField` provides the same constructor overloads as `JTextField`, accepting column count, initial text, and an optional `Document` model. The echo character is initialized to `'*'` by default.

**Beginner-Friendly Explanation.** Creating a password field is just like creating a text field: you decide how wide it should be and whether it should start with any text. The only difference is that the characters are hidden.

---

### Purposes

- To instantiate the password input control with the appropriate preferred width.
- To prepare the field for addition to a login form or authentication dialog.
- To select a column count that gives the user enough room to type comfortably.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JPasswordField field = new JPasswordField();
JPasswordField field = new JPasswordField(20);
JPasswordField field = new JPasswordField("initial", 20);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int columns` | Preferred width in columns. |
| `String text` | Initial text (displayed as echo characters). |

#### Syntax Rules

- The default echo character is `'*'`; no action is needed to enable masking.
- The field is enabled and editable by default.

#### Constraints and Limitations

- Very large column counts may make the field wider than the container.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating Password Fields with Different Constructors

```java
import javax.swing.*;
import java.awt.*;

public class PasswordConstructorDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Password Constructors");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new GridLayout(3, 1, 5, 5));

            JPasswordField empty = new JPasswordField();              // 1. Empty
            JPasswordField sized = new JPasswordField(15);            // 2. Columns
            JPasswordField withText = new JPasswordField("secret", 15); // 3. Text

            frame.add(empty);
            frame.add(sized);
            frame.add(withText);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three password fields: an empty one, a 15-column empty one, and a 15-column field containing six echo characters (for “secret”).

**Why the code produces this result.** Each constructor initializes the field differently. The initial text “secret” is displayed as masked characters.

---

### Real-World Cases

- **Quick empty fields** for login forms where no default is needed.
- **Pre-filled fields** in administrative tools where a password is being reset or viewed.
- **Column-sized fields** to match the layout of a form.

---

## 3. Password Field Methods — getPassword()

### Definitions

**Core Definition.** `getPassword()` returns the password contained in the field as a `char[]` array.

**Technical Definition.** `public char[] getPassword()`. The method retrieves the content of the underlying `Document` and returns it as a character array. The returned array is a copy, so the caller is responsible for clearing it after use. This method replaces the deprecated `getText()` for password retrieval.

**Beginner-Friendly Explanation.** `getPassword()` is how you read the password the user typed. It gives you an array of characters, which you can wipe clean after you are done checking it. You should never use `getText()` on a password field because the resulting `String` cannot be erased from memory.

---

### Purposes

- To securely retrieve the password as a mutable `char[]` that can be cleared from memory.
- To avoid the security risk of `String` objects that linger in memory.
- To provide a standard mechanism for password verification and hashing.
- To comply with security best practices for handling sensitive character data.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
char[] password = field.getPassword();
// use password
Arrays.fill(password, '\0'); // clear
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getPassword()` | Returns the password as a `char[]`. |

#### Syntax Rules

- The returned array is a copy; clearing it does not clear the field’s internal document.
- The array should be cleared immediately after use by setting each element to `'\0'` or `'0'`.
- `getText()` is deprecated and should not be used.

#### Constraints and Limitations

- The field’s internal `Document` still contains the password in its own character array; clearing the returned array does not clear the document.
- No automatic clearing method exists on `JPasswordField` itself.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Password Retrieval and Clearing

```java
import javax.swing.*;
import java.awt.*;
import java.util.Arrays;

public class SecureGetPassword {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Secure getPassword");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPasswordField field = new JPasswordField(20);
            JButton loginButton = new JButton("Login");

            loginButton.addActionListener(e -> {
                char[] password = field.getPassword();               // 1. Get char[]
                try {
                    boolean valid = isValid(password);
                    System.out.println("Valid: " + valid);
                } finally {
                    Arrays.fill(password, '\0');                     // 2. Clear
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(field);
            frame.add(loginButton);
            frame.setSize(350, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static boolean isValid(char[] password) {
        char[] expected = "secret".toCharArray();
        boolean match = Arrays.equals(password, expected);
        Arrays.fill(expected, '\0');
        return match;
    }
}
```

**Expected Output.** Typing “secret” and clicking “Login” prints “Valid: true.” Typing anything else prints “Valid: false.” The character array is cleared in the `finally` block after use.

**Why the code produces this result.** `getPassword()` returns the password as a `char[]`. The comparison uses `Arrays.equals` (not `String.equals`). The `finally` block ensures the password is cleared from memory even if validation throws an exception.

---

### Real-World Cases

- **Login authentication:** Compare the `char[]` against a stored hash or expected value.
- **Password hashing:** Pass the `char[]` to a hashing function (e.g., `MessageDigest`) and clear it afterward.
- **Password strength checking:** Analyze the `char[]` for length, complexity, and common patterns, then clear it.

---

## 4. Password Field Methods — setEchoChar()

### Definitions

**Core Definition.** `setEchoChar(char echo)` sets the character displayed in the password field in place of the actual characters typed by the user.

**Technical Definition.** `public void setEchoChar(char echo)`. The default echo character is `'*'`. Setting the echo character to `'\u0000'` (zero) disables masking entirely, causing the field to display the actual characters as they are typed, similar to a standard `JTextField`. This is a security risk if used unintentionally.

**Beginner-Friendly Explanation.** `setEchoChar` changes what the user sees instead of their password. You can use a dot (`'•'`), a bullet, or any other character. Setting it to zero turns off masking completely, which you should almost never do in production.

---

### Purposes

- To customize the masking character for aesthetic or branding reasons (e.g., using `'•'` instead of `'*'`).
- To temporarily reveal the password for user verification (by setting the echo character to zero), though this must be used with extreme caution.
- To restore masking after a temporary reveal.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
field.setEchoChar('*');      // default
field.setEchoChar('•');      // custom character
field.setEchoChar((char)0);  // disable masking (risky)
char current = field.getEchoChar();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setEchoChar(char)` | Sets the echo character. |
| `getEchoChar()` | Returns the current echo character. |

#### Syntax Rules

- The default echo character is `'*'`.
- Setting the echo character to `'\u0000'` (the null character) disables masking.
- To restore masking, call `setEchoChar` again with a non-zero character.

#### Constraints and Limitations

- Disabling masking exposes the password visually and should never be done in production code without an explicit, user-initiated “show password” toggle.
- Some platforms or look-and-feel implementations may not support all characters.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Echo Character

```java
import javax.swing.*;
import java.awt.*;

public class EchoCharDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Echo Character");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPasswordField field = new JPasswordField(20);
            field.setEchoChar('•');                                  // 1. Bullet

            JButton toggle = new JButton("Show/Hide");
            toggle.addActionListener(e -> {
                if (field.getEchoChar() == (char) 0) {
                    field.setEchoChar('•');                          // 2. Hide
                } else {
                    field.setEchoChar((char) 0);                     // 3. Show
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(field);
            frame.add(toggle);
            frame.setSize(400, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The password field uses a bullet (`•`) for masking. Clicking “Show/Hide” toggles between masked and unmasked display. This is a common “show password” feature.

**Why the code produces this result.** `setEchoChar('•')` sets the custom mask. The toggle checks the current echo character and switches between `'•'` and `'\u0000'` (unmasked). This pattern is acceptable for user-initiated reveal, but the default state must always be masked.

---

#### Example 2: Common Mistake — Disabling Masking Unconditionally

```java
import javax.swing.*;

public class BadEchoChar {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Echo Char");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPasswordField field = new JPasswordField(20);
            field.setEchoChar((char) 0);                             // WRONG: unmasks

            frame.add(field);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The password field displays the actual characters as the user types—there is no masking.

**Why this is wrong.** Setting the echo character to zero disables masking, exposing the password in plain text. This is a security vulnerability that should be avoided in production code.

---

### Real-World Cases

- **Custom UI themes:** Applications that use a specific visual style may set the echo character to match their design.
- **“Show password” toggle:** A controlled, user-initiated reveal that temporarily sets the echo character to zero, then restores it.
- **Accessibility:** Some users may benefit from a larger or more visible mask character.

---

## 5. JPasswordField Best Practices

### Definitions

**Core Definition.** `JPasswordField` best practices are the conventions and patterns that ensure password fields are used securely, correctly, and maintainably in Swing applications.

**Technical Definition.** Best practices include: always using `getPassword()` instead of the deprecated `getText()`; clearing the returned `char[]` after use; never disabling masking in production code without an explicit user action; associating labels with password fields via `setLabelFor` for accessibility; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for password fields are the “rules of thumb” that keep passwords safe: always read the password as a `char[]` and wipe it clean, never show the password by default, and label the field properly for screen readers.

---

### Purposes

- To prevent password exposure through immutable `String` objects.
- To minimize the time sensitive data remains in memory.
- To avoid accidental unmasking of passwords in production UIs.
- To ensure accessibility and keyboard navigation for password fields.
- To maintain thread safety by respecting the EDT.

---

### Syntax Rules and Structure

#### Complete Complete Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JLabel label = new JLabel("Password:");
    JPasswordField field = new JPasswordField(20);                 // 1. Create
    label.setLabelFor(field);                                      // 2. Associate label
    label.setDisplayedMnemonic(KeyEvent.VK_P);                    // 3. Mnemonic

    JButton login = new JButton("Login");
    login.addActionListener(e -> {
        char[] password = field.getPassword();                     // 4. getPassword
        try {
            authenticate(password);
        } finally {
            Arrays.fill(password, '\0');                            // 5. Clear
        }
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use `getPassword()` | Never `getText()`. |
| Clear the `char[]` | `Arrays.fill(password, '\0')` after use. |
| Never disable masking by default | `setEchoChar('\u0000')` only for explicit “show password” actions. |
| Associate label | `label.setLabelFor(field)` for accessibility. |
| Provide mnemonic | `label.setDisplayedMnemonic(KeyEvent.VK_P)`. |
| Respect EDT | `SwingUtilities.invokeLater(...)`. |
| Avoid `String` for passwords | Use `char[]` throughout the authentication pipeline. |

#### Syntax Rules

- **Always use `getPassword()`**, never `getText()`, to retrieve the password.
- **Always clear the returned `char[]`** immediately after use by setting each element to zero.
- **Never set the echo character to zero** unless it is part of an explicit, user-initiated “show password” toggle.
- **Always associate a label** with the password field for accessibility.

#### Constraints and Limitations

- Clearing the returned `char[]` does not clear the field’s internal document; the document still holds the password.
- `JPasswordField` does not provide an automatic `clear()` method.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Login Form

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;
import java.util.Arrays;

public class BestPracticePasswordField {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticePasswordField::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Password");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JLabel label = new JLabel("Password:");
        JPasswordField field = new JPasswordField(20);
        label.setLabelFor(field);                                      // 1. Associate
        label.setDisplayedMnemonic(KeyEvent.VK_P);                    // 2. Mnemonic

        JButton login = new JButton("Login");
        login.addActionListener(e -> {
            char[] password = field.getPassword();                     // 3. getPassword
            try {
                boolean valid = checkPassword(password);
                JOptionPane.showMessageDialog(frame,
                    valid ? "Access granted." : "Access denied.");
            } finally {
                Arrays.fill(password, '\0');                            // 4. Clear
            }
        });

        JPanel panel = new JPanel(new FlowLayout(FlowLayout.LEFT));
        panel.add(label);
        panel.add(field);
        panel.add(login);

        frame.add(panel);
        frame.setSize(450, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }

    private static boolean checkPassword(char[] password) {
        char[] expected = "s3cr3t".toCharArray();
        boolean match = Arrays.equals(password, expected);
        Arrays.fill(expected, '\0');
        return match;
    }
}
```

**Expected Output.** A login form with a labeled password field. Typing the correct password and clicking “Login” shows “Access granted.”; any other password shows “Access denied.” The password `char[]` is cleared after every attempt.

**Why the code produces this result.** The label is associated with the field for accessibility. The password is retrieved as a `char[]`, compared using `Arrays.equals`, and cleared in a `finally` block to ensure it is wiped from memory regardless of the outcome.

---

#### Example 2: Common Mistake — Using `getText()`

```java
import javax.swing.*;

public class BadGetText {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad getText");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPasswordField field = new JPasswordField(20);
            JButton login = new JButton("Login");

            login.addActionListener(e -> {
                String password = field.getText();                    // WRONG: deprecated
                System.out.println("Password: " + password);
            });

            frame.setLayout(new java.awt.FlowLayout());
            frame.add(field);
            frame.add(login);
            frame.setSize(350, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The password is printed to the console as a `String`. The `String` cannot be cleared from memory and may persist until garbage collection.

**Why this is wrong.** `getText()` is deprecated in `JPasswordField` precisely because it returns a `String`, which is immutable and cannot be wiped from memory. The correct approach is to use `getPassword()` and clear the `char[]`.

---

### Real-World Cases

- **Production login dialogs** use `getPassword()` and clear the array immediately after authentication.
- **Password change forms** compare the new password and confirmation as `char[]` arrays and clear both after validation.
- **Security-conscious applications** never disable masking by default and provide an explicit, opt-in “show password” toggle.

---

## References

Oracle. *Class JPasswordField (Java Platform SE 8)*. https://docs.oracle.com/javase/jp/8/docs/api/javax/swing/JPasswordField.html

Oracle. *How to Use Password Fields*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/passwordfield.html

OpenJDK. *JPasswordField (Java Platform SE 8)*. https://cr.openjdk.org/...

Fluid Attacks. *Java JPasswordField Missing Masking*. https://db.fluidattacks.com/rul/SAST-JGAW2/

Stack Overflow. *Revision f615c3d5: JPasswordField getPassword vs getText*. https://stackoverflow.com/revisions/f615c3d5-3b26-4c69-822f-4cedfdc048d2/view-source

OpenJDK Bug Database. *JDK-4288089: getPassword() does not clear intermediate buffer*. https://bugs.openjdk.org/browse/JDK-4288089

Stack Overflow. *Character needed to disable JPasswordField echo not clear*. https://stackoverflow.com/questions/...

GNU Classpath. *Class JPasswordField*. https://developer.classpath.org/sinjdoc/javax/swing/JPasswordField.html